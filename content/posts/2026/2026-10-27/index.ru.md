---
layout: post
title: "Интеграция sandbox Opencode с Intellij IDEA"
aliases: "opencode"
date: 2026-10-27
URL: /ru/2026/10/27/sandbox-opencode-idea-integration-via-acp
tags: [opencode, sandbox, acp, idea]
categories: [Разработка, Software Development]

---

Около года назад я начал активно внедрять AI-агенты (в частности, `opencode`, `kilokode`) в свой рабочий процесс. Как консольные, так и  виде плагинов. Но запускать нейросети, которые имеют прямой доступ к файловой системе и могут выполнять команды в терминале, на хостовой машине - это всегда лотерея. А кликать постоянно "allow" слишком выматывает, когда у тебя три параллельных агента.

В данной статье опишу свой опыт настройки изолированной среды (песочницы) для `opencode`, причины такого решения, подводные камни с путями в IDE и плюсы/минусы конечного сетапа.

## Что не так с запуском на хосте

Немного о проблемах. Запуск агента напрямую в системе имел ряд недостатков, которые со временем начинали меня раздражать:

1. Агентам часто не хватает контекста, они любят покопаться в исходниках зависимостей для лучшего понимания кода. А это доступ во внешнюю директорию и постоянные "Можно про выполнить cat/cd/rm и тому подобное"
2. Хотелось, чтобы агент работал прямо в IntelliJ IDEA через встроенный AI Assistant, но при этом всё равно оставался в изоляции

В общем, хотелось получить надежную изоляцию, как в dev-контейнерах (которая в IDEA только в ultimate), но с прозрачной интеграцией в любимый IDE. Знаю про docker sandbox, но мне не понравилась их политика просить залогиниться для использования песочницы.

## Настройка песочницы

В качестве базы я использую официальный образ `ghcr.io/anomalyco/opencode:latest`, но с небольшой кастомизацией. Все конфигурации лежат у меня в `~/.config/opencode`. 

Ниже приведен `compose.yaml`, который описывает несколько сервисов под разные сценарии использования:

```yaml
x-opencode-settings: &common-opencode-settting
  environment:
    NODE_TLS_REJECT_UNAUTHORIZED: "0"
    OPENCODE_DISABLE_AUTOUPDATE: true
    OPENCODE_DISABLE_DEFAULT_PLUGINS: true
  working_dir: /workspace
  volumes:
    - ~/.config/opencode/agents:/home/opencode/.config/opencode/agents:ro
    - ~/.config/opencode/skills:/home/opencode/.config/opencode/skills:ro
    - ~/.config/opencode/node_modules:/home/opencode/.config/opencode/node_modules:ro
    - ~/.config/opencode/opencode-sbx.jsonc:/home/opencode/.config/opencode/opencode.jsonc:ro
    - ~/.gradle/caches:/home/opencode/.gradle/caches:ro

services:
  opencode:
    image: eshishkin/opencode:latest
    user: "1000:1000"

    build:
      context: .
      dockerfile_inline: |
        FROM ghcr.io/anomalyco/opencode:latest
        RUN apk update && apk add --no-cache curl git
        RUN addgroup opencode && adduser -D opencode -G opencode
        USER opencode
    <<: *common-opencode-settting

  opencode-agent:
    image: eshishkin/opencode:latest
    user: "1000:1000"
    <<: *common-opencode-settting
    entrypoint: opencode run

  opencode-acp:
    image: eshishkin/opencode:latest
    user: "1000:1000"
    stdin_open: true
    init: true
    tty: false
    <<: *common-opencode-settting
    entrypoint: ["opencode", "acp"]
```

По сути, добавил три команды для запуска opencode в разных режимах (терминал, acp и просто one-time прогон) и прокинул необходимые тома в контейнер. Папка `~/.gradle/caches` нужна чтобы агент мог смотреть зависимости и их исходники.

## Алиасы в bash

Писать каждый раз длинные команды `docker compose run ...` быстро надоедает. Поэтому я добавил себе в `.bashrc` несколько алиасов и функций, чтобы работа в терминале не отличалась от работы с локально установленным CLI.

```bash
alias opencode-sbx="docker compose -f ~/.config/opencode/compose.yaml run --remove-orphans --rm -v './:/workspace' opencode"

opencode-sbx-run() {
    if [ -z "$1" ]; then
        echo "Ошибка: Не указана команда для запуска." >&2
        return 1
    fi

    local cmd="$1"
    shift

    docker compose -f ~/.config/opencode/compose.yaml run --remove-orphans --rm -v './:/workspace' opencode-agent --auto "$cmd"
}

opencode-sbx-acp() {
    docker compose -f ~/.config/opencode/compose.yaml run --remove-orphans --rm $@ opencode-acp
}
```
Теперь, находясь в любом проекте, я просто пишу `opencode-sbx` и попадаю в изолированный TUI, а текущая директория автоматически монтируется в `/workspace` внутри контейнера.

## Интеграция с IntelliJ IDEA и проблема путей

Самое интересное началось, когда я захотел подключить этого "песочного" агента к IntelliJ IDEA. Jetbrains сейчас активно развивает протокол APC, который позволяет подключать внешние AI-агенты.

Непредвиденная проблема возникла на этапе инициализации ACP сессии. IDEA запущена на хостовой машине и передает агенту абсолютный путь к проекту именно хостовой машины (например, `/home/eshishkin/projects/my-app`). Но внутри контейнера такого пути нет, понятное дело, нет. Поэтому, хотя IDEA и видела мой агент, но инициализировать она его не смогла.

Решение немного костольные, но работает. Решил дублировать структуру директорий хоста внутри контейнера. То есть, если проект лежит в `/home/eshishkin/projects/my-app`, мы должны примонтировать его именно по этому пути внутрь контейнера.

Так как в конфигурационном файле IDEA (`acp.json`) нельзя просто вызвать `$(pwd)` или использовать переменные окружения для путей монтирования, я написал небольшой bash-скрипт `start-apc-sandbox.sh`, который лежит рядом с `compose.yaml`:

```bash
#!/bin/bash

docker compose -f /home/eshishkin/.config/opencode/compose.yaml run -T --quiet --remove-orphans --rm -v ./:"$(pwd)" opencode-acp
```

Осталось только сказать IDE использовать наш скрипт. В настройках AI Assistant (`acp.json`) я добавил новый agent server в дополнение к обычному не изолированному opencode:

```json
{
  "agent_servers": {
    "OpenCode": {
      "command": "~/.nvm/versions/node/v22.22.0/bin/opencode",
      "args": ["acp"],
      "env": {
        "NODE_TLS_REJECT_UNAUTHORIZED": "0"
      }
    },
    "opencode-sandboxed": {
      "command": "/home/eshishkin/.config/opencode/start-apc-sandbox.sh",
      "args": [

      ]
    }
  }
}
```

Теперь при выборе `opencode-sandboxed` в панели AI IDEA вызывает скрипт, скрипт определяет `pwd`, пробрасывает его в контейнер "один в один" и запускает `opencode acp`.

## Спустя время

По большей части каких-то серьёзных проблем с использованием я не заметил. Все работает более или менее. Из хорошего: я теперь меньше переключаюсь между вкладками и кликаю "Accept". Даю агенту задание и иду заниматься полезными вещеми, не боясь, что ИИ будет творить какую-нибудь дичь. Каждая сессия - отдельный контейнер.

Из минусов: понятное дело, что потребление памяти. Один контейнер отжирает около 300-400 мб. Второй неприятный минус в том, что заметил, что некоторые контейнеры слишком  долго "живут" и удаляются только после закрытия всей IDE, а не только вкладки.

В целом, решение не идеальное, но пока им доволен