---
title: "Инструкция о создании платежного аккаунта для физических лиц в {{ yandex-cloud }}"
description: "Из статьи вы узнаете, как создать платежный аккаунт физическому лицу в {{ yandex-cloud }}. Отвечаем на частые вопросы: платежный аккаунт и платное потребление; стартовый грант; документы."
---

# Начало работы для физических лиц

{% include notitle [start](../_includes/quickstart-start.md) %}

## Создание платежного аккаунта {#new-account}

Платежный аккаунт необходим, даже если вы планируете пользоваться только бесплатными сервисами. При создании первого платежного аккаунта, привязанного к пользовательскому аккаунту, вам будет начислен [стартовый грант](../usage-grant.md).

{% list tabs %}

   - Пробный период

      ![quickstart](../../_assets/overview/individuals-trial-period.svg)

   - Платная версия

      ![quickstart](../../_assets/overview/individuals-paid-version.svg)

{% endlist %}

{% include [main](../../_includes/billing/registration-main.md) %}

Предоставьте данные для создания платежного аккаунта:

1. Укажите ваши имя, фамилию и отчество.

1. Привяжите карту: 

   {% include [pin-card-data](../../_includes/billing/pin-card-data.md) %}
   
   {% include [payment-card-types](../../_includes/billing/payment-card-types.md) %}
   
   {% include [payment-card-validation](../../_includes/billing/payment-card-validation.md) %}

1. Укажите актуальные почту и телефон. Контактные данные нужны не только для связи с вами, но и для выставления счетов и финансовых документов.

1. Если это ваш первый платежный аккаунт в {{ yandex-cloud }}, вам доступно подключение [пробного периода](../free-trial/concepts/quickstart.md).

   {% note info %}

   В некоторых случаях при создании платежного аккаунта с пробным периодом может потребоваться дополнительная верификация. Сообщение с подробной инструкцией появится на странице этого платежного аккаунта в консоли управления.

   {% endnote %}
   
   * Подключая пробный период, помните, что после его завершения ваши ресурсы будут приостановлены. Для возобновления работы потребуется перейти на [платную версию](../free-trial/concepts/upgrade-to-paid.md).
   * Если не подключать пробный период на данном этапе, ваш аккаунт будет создан с платным потреблением: после [использования стартового гранта](../usage-grant.md) вам не придется переходить на платную версию.

1. Нажмите кнопку **Создать**.

{% include [start](../_includes/quickstart-qa-whats-next.md) %}