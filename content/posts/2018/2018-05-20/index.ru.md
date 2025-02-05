---
layout: post
title: "Ликбез по травлению плат или как создать контроллер для автогида"
aliases: "Ликбез по травлению плат или как создать контроллер для автогида"
date: 2018-05-20
URL: /ru/2018/05/20/handmade-auto-guider-for-eq-mounts
tags: 
categories: 

---

Статья не имеет никакой новизны и в большей части ориентирована на меня самого. Поможет мне вспомнить в будущем все детали, если забуду что-нибудь.

Итак, денег у меня на EQ6 нет, и даже на HEQ5 нет, а снимать deepsky хочется. И не по 30 секунд, а поболее.  С блоком EQDrive у меня ничего не вышло - руки не из того места растут, поэтому стал шерстить форум в поисках простого решения. Из самого простого и бюджетного - т.к. "коробочка" от Ивана Ионова (https://astronomy.ru/forum/index.php/topic,19540.0.html). Ну, собственно, решил её сделать сам.

Пришлось немного изменить схему, приведенную на форуме, т.к. микросхему FT245 сейчас днем с огнём не сыщешь. За полтора часика набросал схемку с использованием FT232 в SprintLayout. Первый блин, как говорится, комом и с первого раза ЛУТ-ом не получилось перенести тонер на текстолит из-за того что распечатал на обычной бумаге. На второй раз взял глянец и результат не заставил себя долго ждать.

После того как все зачистил, стал "бодяжить" раствор из хлористого железа. Поначалу пожалел его и медь у меня за час так и не вытравилась до конца. В итоге, после 3 часов, вся ненужная медь ушла, и осталась красивая платка готовая к дальнейшей работе.

{{< gallery >}}
  <img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEg9ao-YxrtobiD8h-FvMB2SdeOo-PZo8cOy9HQF8_2be4ip1bEep5HQb0WbLWRmrcs8K9dwSkIioDmoltR3lF5D3ySEry-IFrYpgJgGJYPi_NHLqpgQG_D4_Ihyta2G9iLHp-k4orlIjNmH/s1600/P1030499.JPG" class="grid-w50 md:grid-w50" />
<img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh3eYt7UgHSQ6PHxziKRWCMg_q1FJRAuFZJCvz2AFA9rhcP2abT1JDp7uo0ECWM7ZBmxMHyUCnSEJOzt-csSqzkJkBQRaX5f1rGlkBEq2N5Fi1EfLWxxdmJB91ilRXIbNsxVT65tVOZVo5v/s1600/P1030500.JPG" class="grid-w50 md:grid-w50" />
<img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEi6Dn9frHzX5ABXmZ3laRHB890Dzd9IM6nZtJ4r4xBjq75C-YEbMRyE35GRsOdWCnMbIla6XBOSa8C1ITSN9C5k0Qbu62kkpsDIscgn30adiQZ8YfGqmH_7jHv8QdT8VCdQPReZ4FB5TRzA/s1600/P1030501.JPG" class="grid-w100 md:grid-w100" />
{{< /gallery >}}

Поскольку дрели у меня не было, пришлось использовать то, что было. По бюджетному же :). А был у меня набор надфилей из набора для моделизма.  

{{< gallery >}}
<img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEioQGTkdlS40yoQtMP93XDZI_V6h16L7H4D_O_VrgA4KqiUr4ccAXefGvGMlUhv_p4gxghKzV5pFfAmJ1KPd5k8ahjoFcwFUCsQnamvTNVgN2UmoclK7nWK0_7sZNRq55r6FdowfR88Rp-X/s1600/P1030504.JPG" class="grid-w100 md:grid-w100" />
{{< /gallery >}}

  
Очистил плату от тонера, проверил на замыкания, залудил, начал паять компоненты и... Было много и много мата, т.к. неправильно развёл дорожки для FT232.  
  
Спустя три дня повторил все описанные выше шаги  опять. Второй раз было намного легче. Когда все было готово, обжал отвёрткой кабель и подпаял его к контактам пульта монтировки. Немного настроек ASCOM драйвера и "гоу-ту" готов. Получилась такая платка.  

{{< gallery >}}
  <img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEj54Tfc3TuQEE3Wx6kvgHdk_A1GcUdseIvECWzM0qw63XtcppCNzkzM7bkhsBw41S7PHB_CnH5J9tvYp-qlHoY9tvnhwTgmUbH4Qlgywk59en0Qo6QErTxphNEDHumXmwkt6deWxIYkyhYl/s1600/P1030509.JPG" class="grid-w50 md:grid-w50" />
<img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiyebGZsKEJOSLupMzrY8KxoyAHqvtKq5fv0x5Yapr1qrAlELXP7nPi_05vbdBGre303QeIbLY-XZZyN3LrzMP4s4lweYkJs4l4fmEDCUzPcdrJew6Ba92cG0D7Agitaq19Q-tZmZESHdnI/s1600/P1030511.JPG" class="grid-w50 md:grid-w50" />
<img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhiv8ozTx4ZGBM5JL-cIv_jEO1HXxoy_LI15trT5AeEDwg6H5NDkwerTucFH8loZuAXVb0SbFok0iG8KU2MNMo_MwqD3qjtAhx7Z7zg2zwwxhbHx3mWZkSh_yKdEmTtlLdPOAAV8FDeaoWu/s1600/P1030508.JPG" class="grid-w100 md:grid-w100" />
{{< /gallery >}}

Сам "сетап" выглядит так.
  
{{< figure
    src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgjg24MDq3hQx2HYWIakxAVxKJXvyXkoH8uyu_x51wVyJtnJ_EkaOjapeZuzlbIR_RUmPqYIZG8NHaZNew91A3WVoqsaMDoZw11E5y5aanKv4-NtSxShPLyq35L3yjoUxA9f_Bh0itJCa8E/s1600/P1030516.JPG"
    alt="Sky Watcher 150/1200 + EQ5"
    caption="Sky Watcher 150/1200 + EQ5"
>}}

Гидирование пока не проверял, но через планетарий "Cartes du Ciel", монтировка наводится довольно точно на близких дистанциях. Пробовал от Веги до M57 ("Кольцо"). Туманность была недалеко от центра поля зрения 10 мм окуляра. Довольно хороший результат, на мой взгляд. Скоро попробую что-нибудь поснимать.