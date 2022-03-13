HTB (intelligence)より

# Tool
https://github.com/dirkjanm/krbrelayx

# 実際の例

[使い方と説明](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/adidns-spoofing)

## query a node

![winsforwardlookup](https://user-images.githubusercontent.com/85237728/158052284-5f64fbc2-aa78-43bd-bbd9-8894a1fb032d.png)

## add a node and attach a record

![addrecord](https://user-images.githubusercontent.com/85237728/158052614-8625d1ec-97a9-4e4d-8f7f-6fda8e8f98ef.png)

いま作ったレコードを問い合わせてみたところ：

![newrecord](https://user-images.githubusercontent.com/85237728/158052677-146e485a-ea75-469b-b49f-dce696619f3e.png)

# レコードのダンプには`adidnsdump`

![recorddump](https://user-images.githubusercontent.com/85237728/158055339-e4d87b10-ff68-48fb-90b8-41425177c9b7.png)

-> 「type」ではなく「name」がレコード名（上の一行目の「type,name,value」より）
