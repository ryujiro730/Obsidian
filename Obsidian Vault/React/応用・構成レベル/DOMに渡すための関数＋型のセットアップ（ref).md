まずconst 関数名で関数名を名付ける。
その後に＝をつけて[[forwardRef]]<any,Props>→このanyは[[Ref]]の型で、[[props]]は[[props]]の型。

その後、function 関数名（…）→ここが関数の本体。
==**ここで注意**==
ここで最初に[[const]]で関数名を宣言してるのにどうしてまたfunctionでも関数名を定義してるの？って聞くと、それあデバッグの時に誰がエラーを出したかわかるようにするため。


その後に引数を書く。({ selectedSymbol, selectedTimeframe, chartHeight }, ref)こんな感じ