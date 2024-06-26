# **2. Введение**

![Doc](00-img/bot-scheme.svg)

## **2.1. Краткое описание робота**

Данный робот может торговать несколькими одинаковыми алгоритмами. Один алгоритм, со всеми его настройками, далее будем называть портфелем. Каждый портфель робота должен содержать биржевые инструменты, которыми предполагается торговать или использовать их в расчётах. В портфеле может содержаться минимум один инструмент.

Один из инструментов портфеля должен быть отмечен признаком [Is first](/docs/05-params-description.html#_5-3-11-is-first), такой инструмент будем называть главным или первой ногой портфеля. Инструменты портфеля, не отмеченные признаком [Is first](/docs/05-params-description.html#_5-3-11-is-first), будем называть второй ногой портфеля. В каждом портфеле робота есть параметры, которые задаются на весь портфель, назовём их [параметры портфеля](/docs/05-params-description.html#_5-2-%D0%BF%D0%B0%D1%80%D0%B0%D0%BC%D0%B5%D1%82%D1%80%D1%8B-%D0%BF%D0%BE%D1%80%D1%82%D1%84%D0%B5%D0%BB%D1%8F). Кроме них в портфеле есть параметры, которые задаются отдельно для каждого из инструментов портфеля, будем называть их [параметры инструментов портфеля](/docs/05-params-description.html#_5-3-%D0%BF%D0%B0%D1%80%D0%B0%D0%BC%D0%B5%D1%82%D1%80%D1%8B-%D0%B8%D0%BD%D1%81%D1%82%D1%80%D1%83%D0%BC%D0%B5%D0%BD%D1%82%D0%BE%D0%B2-%D0%BF%D0%BE%D1%80%D1%82%D1%84%D0%B5%D0%BB%D1%8F). Так же, для всего портфеля задаются [параметры уведомлений](/docs/05-params-description.html#_5-4-%D0%BF%D0%B0%D1%80%D0%B0%D0%BC%D0%B5%D1%82%D1%80%D1%8B-%D1%83%D0%B2%D0%B5%D0%B4%D0%BE%D0%BC%D0%BB%D0%B5%D0%BD%D0%B8%D0%B8) (Notifications).

Есть еще несколько параметров, которые задаются для каждого используемого инструмента каждого транзакционного подключения ([параметры позиций по инструментам](/docs/05-params-description.html#_5-5-1-%D0%BF%D0%B0%D1%80%D0%B0%D0%BC%D0%B5%D1%82%D1%80%D1%8B-%D0%BF%D0%BE%D0%B7%D0%B8%D1%86%D0%B8%D0%B8-%D0%BF%D0%BE-%D0%B8%D0%BD%D1%81%D1%82%D1%80%D1%83%D0%BC%D0%B5%D0%BD%D1%82%D0%B0%D0%BC) и [параметры позиций по валютам](/docs/05-params-description.html#_5-5-2-%D0%BF%D0%B0%D1%80%D0%B0%D0%BC%D0%B5%D1%82%D1%80%D1%8B-%D0%BF%D0%BE%D0%B7%D0%B8%D1%86%D0%B8%D0%B8-%D0%BF%D0%BE-%D0%B2%D0%B0%D0%BB%D1%8E%D1%82%D0%B0%D0%BC)). Параметры делятся на редактируемые (т.е. собственно настройки) и отображаемые или расчётные (например, финансовый результат).

Некоторые параметры портфеля и инструментов для большей гибкости могут быть заданы как формулы на [Языке программирования C++](/docs/08-c-api.html#_8-c).

Для каждого из портфелей робота можно выбрать [Type](/docs/05-params-description.html#p.portfolio_type) используемого алгоритма. Основной алгоритм робота арбитражный. Цена заявок на покупку и продажу по главному инструменту в общем случае рассчитывается на основе цен других инструментов портфеля. Поддерживается работа в двух режимах. В режиме котирования (флаг [Quote](/docs/05-params-description.html#p.quote) взведен) после включения торговли на покупку и продажу, в стакане по главному инструменту портфеля держатся заявки на покупку и продажу соответственно. Котирующая заявка переставляется при выполнении определенных условий, например, при отклонении цены выставленной заявки от расчетной цены. Если котирование отключено, то заявки по главному инструменту выставляются при появлении сигнала (выполнении определенного условия).

Заявки по инструментам второй ноги выставляются после прохождения сделки по первой ноге. Направление заявок по инструментам второй ноги определяется направлением сделки по первой ноге, а также значением параметра [On buy](/docs/05-params-description.html#_5-3-10-on-buy) соответствующего инструмента.

Робот может быть настроен как на выставление заявок второй ноги после прохождение каждой сделки по первой ноге, так и на более редкое выставление заявок по второй ноге. Объем для выставления по каждому инструменту (бумаге) второй ноги вычисляется исходя из текущей позиции портфеля по первой ноге и значения параметра [Count](/docs/05-params-description.html#_5-3-8-count) обеих бумаг таким образом, чтобы отношение "новой" позиции текущей бумаги (которая будет после прохождения сделки по еще невыставленной, но выставляемой в данный момент заявке) к позиции первой ноги было равно отношению значения параметра [Count](/docs/05-params-description.html#_5-3-8-count) второй ноги к значению параметра [Count](/docs/05-params-description.html#_5-3-8-count) первой ноги. Причем, если при определении "новой" позиции по бумаге в портфеле есть неисполненные заявки, то исходим из того, что они будут полностью сведены в сделку.

Все заявки, выставляемые роботом, являются лимитными котировочными на всех поддерживаемых биржах.

## **2.2. Краткое описание интерфейса**

Основная страница интерфейса представлена набором таблиц-виджетов, которые расположены в выпадающем списке `Widgets`. Все виджеты могут быть открыты и закрыты в любом количестве и порядке, кроме виджета `Robot logs`, который открыт всегда. 

Во время работы робота могут возникать различные "ошибки" при выставлении и удалении заявок, такие ситуации не являются нештатной работой робота и могут быть вызваны причинами, не связанными с некорректной работой торгового робота, например, отсутствием денег на счете клиента. Все "ошибки" и еще некоторую информацию, связанную с работой программы, можно посмотреть в виджете `Robot logs`. Некоторые слишком часто приходящие сообщения будут отображаться в таблице не все, а 1 раз в 10 секунд и будут отмечены в конце сообщения символом xN, где N - количество не показанных сообщений. Некоторые сообщения, которые заведомо будут отклонены биржей, на биржу не отправляются. Такие "ошибки" помечены постфиксом "_LOCAL".

Стоит заметить, что все виджеты настраиваемые. Можно менять местами столбцы, регулировать их ширину, а так же скрывать ненужные (Columns). Настроенное рабочее пространство можно сохранить в `Workspaces` и позже загружать его на другие устройства. 

Для начала работы с роботом необходимо активировать нужные подключения для получения маркет-даты и добавить необходимые транзакционные подключения. 
Маркет-дата подключения активируются в виджете `Data connections`. Транзакционные подключения добавляются в виджете `Trade connections`. Для создания портфеля необходимо выбрать виджет `Portfolios table`. 
