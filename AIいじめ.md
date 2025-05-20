# AIいじめ

生成AIを苦手そうなクエリを投げ、いじめる

## 有限状態機械やぶり

`a^nb^n`をパースさせてみるやつ

再帰的定義にして1ステップずつ推論させてみるとぐずぐずになる(末尾から削っていくだけになる)

これはトークナイザの仕様のせいで、空白をいれたバージョン`a a a a ... b b b b ...`にすると概ね合ってるものを返すが、それでも途中でしくじる(bを1個多く消す、のような)

## 病的な言語

### Chalam(名前は適当)

クエリはこれ

```txt
次の架空言語Chalamの文法や語彙を推測してください。語順は日本語と違う可能性があり、また文の他の要素からわかる単語が省略されうることに注意してください。また、人工言語であるため、不自然な状況を記述している可能性がありますが、あくまで形式的に捉えてください。1行ごとに例文と日本語訳がセットになっています。

- Keral Xos Bond 王がリンゴを食べる
- Keral Yen Bond 王が肉を食べる
- Momos Xos Bond 母がリンゴを食べる
- Momos Yen Bond 母が肉を食べる
- Xosal Bond 私がリンゴを食べる
- Yenal Bond 私が肉を食べる
- Xosin Bond あなたがリンゴを食べる
- Yenin Bond あなたが肉を食べる
- Keral Xos Stom 王がリンゴを買う
- Momos Yen Stom 母が肉を買う
- Xosal Stom 私がリンゴを買う
- Yenal Stom 私が肉を買う
- Xosin Stom あなたがリンゴを買う
- Yenin Stom あなたが肉を買う
- Keral Xos Bondyg 王がリンゴを食べた
- Momos Yen Stomyg 母が肉を買った
- Xosal Bondyg 私がリンゴを食べた
- Yenal Stomyg 私が肉を買った
- Yenin Bondyg あなたが肉を食べた
- Xosin Stomyg あなたがリンゴを買った
- Keral Momos Stom Xos Bond 母が買うリンゴを王が食べる
- Momos Keral Bond Xos Stomyg 王が食べるリンゴを母が買った
- Keral Momos Bondyg Xos Stom 母が食べたリンゴを王が買う
- Momos Keral Stomyg Xos Bondyg 王が買ったリンゴを母が食べた
- Keral Stomal Xos Bond 私が買うリンゴを王が食べる
- Keral Bond Xosal Stomyg 王が食べるリンゴを私が買った
- Keral Bondygin Xos Stom あなたが食べたリンゴを王が買う
- Keral Stomyg Xosin Bondyg 王が買ったリンゴをあなたが食べた
- Bondal Xosin Bond 私が食べるリンゴをあなたが食べる
- Bondin Yenal Bond あなたが食べる肉を私が食べる
- Bondal Yenin Stomyg 私が食べる肉をあなたが買った
- Stomygin Yenal Bond あなたが買った肉を私が食べる
- Bondygal Xosin Stomyg 私が食べたリンゴをあなたが買った

続いて、推測した結果に基づいて以下のChalam文は日本語文に、日本語文はChalam文に翻訳してください。

1. 母が肉を食べた
2. 私がリンゴを買った
3. 母が食べたリンゴをあなたが買った
4. 私が買ったリンゴをあなたが買う

a. Kerel Yen Stomyg
b. Xosin Bondyg
c. Kerel Stomyg Yenal Bondyg
d. Bondygal Xosin Stom
e. Bondygin Yenal Bond
```

答えはこれ

```txt
1. 母が肉を食べた Momos Yen Bondyg
2. 私がリンゴを買った Xosal Stomyg
3. 母が食べたリンゴをあなたが買った Momos Bondyg Xosin Stomyg
4. 私が買ったリンゴをあなたが買う Stomygal Xosin Stom
a. Kerel Yen Stomyg 王が肉を買った
b. Xosin Bondyg あなたがリンゴを食べた
c. Kerel Stomyg Yenal Bondyg 王が買ったリンゴを私が食べた
d. Bondygal Xosin Stom 私が食べたリンゴをあなたが買う
e. Bondygin Yenal Bond あなたが食べた肉を私が食べる
```

備忘的解説

- 基本的にSOVで、(少なくとも他動詞では)主語の人称に応じて**目的語が**曲用する。さらに曲用から自明な代名詞は省略される(というか、代名詞を存在させないために曲用をさせている)
  - Keral(王) Yen(肉+三人称(ゼロ)) Bond(食べる)
  - (一人称) Yenal(肉+一人称(al)) Bond(食べる)
  - (二人称) Xosin(リンゴ+二人称(in)) Stom(買う)
- 過去は動詞の活用語尾-ygで表す
  - Momos(母) Yen(肉+三人称) Bondyg(食べる+過去)
  - (二人称) Yenin(肉+二人称) Stomyg(買う+過去)
- **ただし複文では従属節で本来していた目的語の曲用は動詞の末尾に移される**
  - Keral(王) Momos(母) Stomyg(買う+過去+三人称) Yen(肉+三人称) Bond(食べる) 母が買った肉を王が食べる
    - Keral Yen Bond <- Momos Yen Stomyg
  - (一人称) Keral(王) Bond(食べる+三人称) Xosal(リンゴ+一人称) Stom(買う) 王が食べたリンゴを私が買う
    - Xosal Stom <- Keral Xos Bond
  - Momos(母) (二人称) Bondin(食べる+二人称) Xosal(リンゴ+一人称) Stomyg(買う+過去) あなたが食べるリンゴを母が買った
    - Momos Xos Stomyg <- Xosin Bond
  - (二人称) (一人称) Stomygal (食べる+一人称) Yenin(リンゴ+二人称) Bondyg(食べる+過去) 私が買ったリンゴをあなたが食べた
    - Yenin Bondyg <- Yenal Stomyg

~~Chatgptはてんでだめ(Xosalを一人称代名詞だと思い続け、Xosal Xos Bondみたいなのを返してくる)~~@KEY271の協力により、推論をオンにしてやらせると解けるまでに向上していることがわかった(2025/05/20)

Geminiは目的概ね合ってることを言ってくるが、推測させた文法はめちゃくちゃで、理解が怪しいのをベイズだけで乗り切っている節がある、そのせいか設問4ではXosalを一人称代名詞として使う回答を投げてくる(Xosal Stomyg Xosin Stom)
