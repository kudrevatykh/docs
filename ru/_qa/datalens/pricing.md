# Тарификация и оплата

### Новые возможности сервиса будут появляться только в тарифе Business? {#new-features-on-business}

Нет, новые базовые возможности сервиса будут доступны во всех [тарифах](../../datalens/pricing.md). Но некоторые отдельные обновления будут доступны только в платном тарифе.

### Можно ли остаться на старом {{ datalens-short-name }}, без тарифов? {#use-datalens-without-pricing}

23 апреля 2024 года все экземпляры {{ datalens-short-name }} будут переведены на бесплатный тарифный план Community. Вы можете остаться на нем, если ваши пользователи используют аутентификацию с [Яндекс ID](https://yandex.ru/support/id/index.html) / [Яндекс 360](https://yandex.ru/support/business/index.html). При этом никаких ограничений для вас не будет.

Если у вас была настроена аутентификация через [федерацию удостоверений](../../organization/concepts/add-federation.md) (корпоративный SSO, например через LDAP), то до 31 декабря 2024 года ваш экземпляр {{ datalens-short-name }} будет работать на бесплатном тарифном плане Community. Чтобы продолжить использование {{ datalens-short-name }}, вам нужно запланировать переход на тарифный план Business или на использование Яндекс ID.

### Каков график введения тарификации? {#timeline}

Введение тарификации сервиса {{ datalens-short-name }} будет поэтапным:

* 20 марта 2024 года — анонс тарифов и оповещение всех пользователей;
* 23 апреля 2024 года — можно будет перейти на тарифный план Business бесплатно;
* 1 июня 2024 года — тарифный план Business становится платным;
* 31 декабря 2024 года — для старых пользователей SSO перестает работать на бесплатном тарифе Community.

### Можно ли вернуться на бесплатный тариф Community? {#free-rollback}

Да. Переход возможен только с начала следующего месяца. При этом возможности платного тарифа будут отключены.

### Можно ли частично перейти на Business? Например, 10 пользователей Business, остальные — Community {#part-business}

Нет, переход происходит для всей [организации](../../datalens/concepts/organizations.md), в которой активирован {{ datalens-short-name }}.

### Что делать, если есть как федеративные пользователи, так и обычные? {#sso-and-users-pricing}

Если важно сохранить работу федеративных пользователей, запланируйте переход на тарифный план Business. При этом учтите, что на этом тарифном плане тарифицируются все пользователи — и SSO, и Яндекс ID.

### Как будут считаться пользователи публичных чартов/дашбордов? {#public-pricing}

[Публичные ссылки](../../datalens/concepts/datalens-public.md) на дашборды и чарты работают без аутентификации и при подсчете активных пользователей не учитываются. Публичные чарты и дашборды будут бесплатны на всех тарифах.

