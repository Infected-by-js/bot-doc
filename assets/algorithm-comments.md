---
title: 8. Примечания к алгоритму
section: 8
---

# Примечания к алгоритму

На данной странице собраны подробные описания неочевидных моментов работы алгоритма робота.

## Особенности использования торговых стаканов

В роботе для получения цен используются таблицы/потоки торговых стаканов по инструментам. Особенность работы со стаканами такова, что стакан не приходит с биржи в готовом виде. Чтобы минимизировать объём пересылаемой информации биржа присылает сначала снимок/слепок стакана по инструменту на какой-то момент времени, а потом приходят только обновления, сообщающие об изменениях в стакане. Робот НЕ строит стакан сразу по всем бумагам, стакан строится только по используемым в портфелях бумагам. Поэтому, для добавления новой бумаги в список бумаг, по которым строится стакан, необходимо переоткрыть поток с текущим слепком, а потом применять к нему обновления из потока инкрементальных обновлений. В момент получения потока со слепком, стакан в роботе временно перестает обновляться по всем используемым бумагам. Он начнет обновляться только после того как будет закрыт поток со слепком (это может занять некоторое время, зависящее от общего количества бумаг, приходящих в данном потоке, и размера стаканов) и начнут применяться инкрементальные обновления. Соответственно, в момент переоткрытия стакана вы можете наблюдать отсутствие цен по бумагам. Переоткрытие требуется в большинстве подключений, потому что стаканы по всем бумагам или полный лог заявок приходят, как правило, в одном потоке, т.е. нельзя получать данные по конкретным бумагам, получать вы всегда будете все данные, а использовать можете только для нужных бумаг.

Стакан в роботе может переоткрываться в следующих случаях:

- создание нового портфеля;
- добавление бумаги в портфель;
- снятие флага `Disabled` с портфеля;
- пропуски в номерах сообщений инкрементального потока обновлений стакана в UDP подключениях к биржам (чем больше у вас портфелей и бумаг, тем выше вероятность возникновения этих пропусков).

Т.е. произойдёт приостановка торговли по всем портфелям, использующим инструменты данной биржи. Это не является ошибочным поведением робота.

## Использование приказа переместить заявку

На некоторых подключениях реализована отправка приказа переместить заявку (иногда эта команда также называется изменением заявки). Использование данного приказа позволяет сократить количество транзакций и увеличить отношение количества сделок к количеству транзакций, особенно при котировании, а так же увеличить время нахождения заявок в рынке. Так как особенности использования данной команды отличаются на разных рынках и типах подключений, то, в зависимости от подключения, данный приказ может применяться только для первой ноги портфеля или же для инструментов обеих ног. На данный момент отправка такой команды реализована для FIX-подключений фондового и валютного рынков Московской биржи, а так же TWIME-подключения срочного рынка Московской биржи.

Использование приказа переместить заявку для поддерживаемых подключений не является обязательным. Эта возможность может быть отключена при создании нового транзакционного подключения.

## Правила перемещения Lim_Sell и Lim_Buy

Сигнальные цены [Lim_Sell](params-description.md#p.lim_s) и [Lim_Buy](params-description.md#p.lim_s) перемещаются только при прохождении сделок по [Is first](params-description.md#s.is_first) инструменту портфеля кроме случая использования [Always timer](params-description.md#p.always_limits_timer).

Правила перемещения сигнальных цен можно разделить на две части: произошла продажа по [Is first](params-description.md#s.is_first) бумаге и произошла покупка по [Is first](params-description.md#s.is_first) бумаге. Внутри каждой из этих частей алгоритм делится еще на две части: позиция портфеля до прохождения данной сделки была равна нулю и была не равна нулю.

Введем следующие обозначения: `diffpos` - знаковое количество лотов в сделке по [Is first](params-description.md#s.is_first) бумаге, `V` - это [v_in_left](params-description.md#p.v_in_l) × [Count](params-description.md#s.count) или [v_out_left](params-description.md#p.v_out_l) × [Count](params-description.md#s.count) в зависимости от того открываем мы позицию или закрываем данной заявкой, [Count](params-description.md#s.count) - это `Count` [Is first](params-description.md#s.is_first) бумаги, [Curpos](params-description.md#s.pos) - текущая позиция по [Is first](params-description.md#s.is_first) бумаге портфеля (т.е. прошедшая только что сделка еще НЕ учтена), нижний индекс 0 - предыдущее значение параметра, 1 - новое значение параметра. В данных обозначениях алгоритм перемещения сигнальных цен примет вид:

- если прошла продажа(соответственно в количестве `diffpos`):
    
    - если текущая позиция до момента совершения сделки была $curpos\neq 0$, то:

        $k3=\left(|{Lim\_Sell_0- Lim\_Buy_0}|-TP-K\right)\times\frac{V}{curpos},$

        $k4=
  \begin{cases}k3+K2, &\text{if}\enspace Lim\_Sell_0-Lim\_Buy_0\geq 0\\
              -k3+K2, &\text{if}\enspace Lim\_Sell_0-Lim\_Buy_0<0 
  \end{cases},$ 

        $Lim\_Buy_1= Lim\_Buy_0+\frac{|{diffpos}|}{V}\times 
    \begin{cases} 
       k4, &\text{if}\enspace curpos>0\\ 
       K1, &\text{if}\enspace curpos<0 
    \end{cases},$

        $Lim\_Sell_1=Lim\_Sell_0+\frac{|{diffpos}|}{V}\times
   \begin{cases} 
     K2, &\text{if}\enspace curpos>0\\ 
      K, &\text{if}\enspace curpos<0 
   \end{cases},$

    - если текущая позиция до момента совершения сделки была $curpos=0$, то:

        $Lim\_Sell_1=Lim\_Sell_0+\frac{|{diffpos}|}{V}\times K,$

        $Lim\_Buy_1=Lim\_Sell_0-TP,$

- если прошла покупка (соответственно в количестве diffpos):

    - если текущая позиция до момента совершения сделки была $curpos\neq 0$, то:

        $k3=\left(|Lim\_Sell_0-Lim\_Buy_0|-TP-K\right)\times\frac{V}{curpos},$

        $k4=
  \begin{cases} 
    -k3+K2, &\text{if}\enspace Lim\_Sell_0-Lim\_Buy_0\geq 0\\
     k3+K2, &\text{if}\enspace Lim\_Sell_0-Lim\_Buy_0<0
  \end{cases},$
	
        $Lim\_Sell_1=Lim\_Sell_0-\frac{|{diffpos}|}{V}\times 
   \begin{cases} 
     k4, &\text{if}\enspace curpos<0\\
     K1, &\text{if}\enspace curpos>0 
   \end{cases},$
   	
        $Lim\_Buy_1=Lim\_Buy_0-\frac{|{diffpos}|}{V}\times 
   \begin{cases} 
     K2, &\text{if}\enspace curpos<0\\
      K, &\text{if}\enspace curpos>0 
   \end{cases},$

    - если текущая позиция до момента совершения сделки была $curpos=0$, то:
	
        $Lim\_Sell_1=Lim\_Buy_0+TP,$
	
        $Lim\_Buy_1=Lim\_Buy_0-\frac{|{diffpos}|}{V}\times K.$

Также перемещение сигнальных цен происходит когда заявка не может быть выставлена из-за ограничений по [v_min](params-description.md#p.v_min), [v_max](params-description.md#p.v_max), [To0](params-description.md#p.to0). Если робот не может купить из-за ограничений по [v_max](params-description.md#p.v_max), то в соответствии с параметрами портфеля [Limits timer](params-description.md#p.timer) и [Percent](params-description.md#p.percent) цены [Lim_Sell](params-description.md#p.lim_s) и [Lim_Buy](params-description.md#p.lim_b) уменьшаются на величину параметра портфеля [K](params-description.md#p.k), если же робот не может продать из-за ограничений по [v_min](params-description.md#p.v_min), то в соответствии с параметрами портфеля [Limits timer](params-description.md#p.timer) и [Percent](params-description.md#p.percent) значения [Lim_Sell](params-description.md#p.lim_s) и [Lim_Buy](params-description.md#p.lim_b) увеличиваются на величину параметра портфеля [K](params-description.md#p.k).

## Особенности поведения заявок, переставленных по SL или Timer <Anchor :ids="['sl_timer']" />

При нулевом или положительном значении параметра [k_sl](params-description.md#s.k_sl) заявки, выставленные по следующим событиям: переставление заявки из-за срабатывания условия на [SL](params-description.md#s.sl), переставление заявки из-за срабатывания условия на [Timer](params-description.md#s.timer), закрытие или выравнивание позиции в соответствии с настройками в расписании или по нажатию на "кликер" [To market](params-description.md#p.to_market), будут переставляться раз в секунду до момента сведения в сделку, до выключения торговли с помощью [Hard stop](getting-started.md#portfolio_actions.hard_stop) или до получения ошибки выставления. Переставление заявок будет осуществляться на цену `bid` - [k_sl](params-description.md#s.k_sl) для продажи и `offer` + [k_sl](params-description.md#s.k_sl) для покупки.

Это дополнительное переставление, оно не меняет существующие настройки параметра [Timer](params-description.md#s.timer) и параметра [TE](params-description.md#s.te) и не зависит от них, т.е. оно выполняется даже при снятом флаге [TE](params-description.md#s.te).

## Подсчёт финансового результата

Финансовый результат (в смысле просто число) считается по сделкам и никаких "экзотических" случаев, связанных с его подсчетом нет. НО есть случаи когда не будет раздвижки в таблицах виджетов финансовых результатов ([Finres for today](interface.md#finres_for_today) и [Finres history](interface.md#finres_history)). Нужно запомнить главное правило, чтобы была раздвижка, должна быть сделка по главной бумаге, если ее нет, то и раздвижки нет. То есть, если у вас по какой-то причине "кривая" позиция и вы ее "подровняли", нажав на кнопку [To market](params-description.md#p.to_market), то вы не получите нормальной раздвижки в данных виджетах. У вас будет только одна "кривая" раздвижка, в которой фигурирует только главная бумага (как пример, флуд контроль, проходят сделки по первой ноге, а по второй не дают выставиться, у вас будут одноногие "кривые" раздвижки с первой ногой, а после нажатия To market, когда пройдут сделки, т.к. они были только по второй ноге, то раздвижки в таблице не будет, но с финансовым результатом все будет в порядке).

И еще один вариант, когда значение параметра [Count](params-description.md#s.count) первой ноги больше значения параметра  [Count](params-description.md#s.count) второй. Пусть вы торгуете долларом валюты против доллара на срочном рынке, и валюта первая нога. Т.е. [Count](params-description.md#s.count) у валюты 100, а у срочки 1, и вы перекрываете на срочке каждые 100 контрактов валютки. Вы выставили 100 контрактов на валютке. Предположим, у вас взяли 60. Раздвижки в таблице не будет, т.к. она получится заведомо "кривой", вторую ногу же не кидали. Потом прошло еще 50, и вы снова не увидите раздвижки. Да, вы кините одну бумагу на срочку, она пройдет (в финансовом результате все нормально учтется). Но опять же не совсем ясно к каким сделкам ее привязывать, если привязать как обычно к последней (т.е. к 50), то будет заведомо кривая раздвижка, а искать какие-то предыдущие сделки уже не вариант, т.к. все могло быть не так просто, как в описанном примере.

## О ценах выставления заявок второй ноги

Заявки по инструментам второй ноги выставляются по следующим ценам: заявка на покупку выставляется по лучшей цене на продажу, к которой прибавляется отступ [k](params-description.md#s.k) или отступ [k_sl](params-description.md#s.k_sl) в случае переставления заявки по стоп лоссу и другим аналогичным событиям, заявка на продажу выставляется по лучшей цене на покупку, из которой вычитается отступ [k](params-description.md#s.k) или отступ [k_sl](params-description.md#s.k_sl) в случае переставления заявки по стоп лоссу и другим аналогичным событиям. При этом лучшие цены на покупку и продажу по инструментам второй ноги фиксируются в момент выставления заявки по [Is first](params-description.md#s.is_first) инструменту. Таким образом, в момент совершения сделки по [Is first](params-description.md#s.is_first) инструменту все остальные бумаги в портфеле выставятся с отступами не от текущих лучших цен, а от лучших цен, которые были во время выставления заявки по [Is first](params-description.md#s.is_first) инструменту. Исключение составляет использование параметра [Equal price](params-description.md#p.equal_prices).

Других вариантов вариантов формирования цен заявок второй ноги, кроме описанных выше, не предусмотрено.

## Об объёмах выставления заявок

Для некоторых бирж (к примеру Deribit) если объём в заявке не кратен лоту, то объем выставляемой заявки принудительно округляется до целого количества лотов(например лот равен 10, объём выставляемой заявки 25, то в действительности на бирже выставится заявка объемом 20 (округление всегда происходит в меньшую сторону)). Если при подобном округлении объём выставляемой заявки станет равным нулю, то вы получите ошибку выставления `REASON_ZERO_AMOUNT_TO_MULTIPLE`.
