---
type: slides
---

# 単語ベクトルと意味的類似度

Notes: このレッスンではspaCyを使って、文章やスパン、トークン間の類似度を予測する方法を学んでいきます。

そして、単語ベクトルとは何か、それをどのようにNLPアプリケーションに使っていくかということも学んでいきます。

---

# 意味的類似度を比較する

- `spaCy`を使うと、2つのオブジェクトを比較、類似度を予測することができます
- `Doc.similarity()`、`Span.similarity()`、`Token.similarity()`
- 他のオブジェクトに対して、類似度（0~1）を返します
- **重要：**単語ベクトルが含まれているモデルが必要です。例：
  - ✅ `en_core_web_md` （中サイズモデル）
  - ✅ `en_core_web_lg` （大サイズモデル）
  - 🚫 `en_core_web_sm`は非対応（小サイズモデル）

Notes: spaCyを使うと、2つのオブジェクトを比較し、類似度を予測できます。例えば、doc、span、tokenなどです。

`Doc`と`Token`と`Span`の`.similarity`メソッドは、他のオブジェクトを引数にとり、類似度を示す0以上1以下の小数点数を返します。

重要：類似度を取得するには、単語ベクトルが入っている大きなモデルを使用する必要があります。

例えば、中もしくは大サイズの英語モデルです。小サイズのものは対応していません。
なので、もし単語ベクトルを使いたいときは、名前が「md」もしくは「lg」で終わっているモデルを使ってください。
詳細はこちらをご覧ください：[models documentation](https://spacy.io/models)。

---

# 類似度の例(1)

```python
# 単語ベクトルが含まれる大きなモデルをロード
nlp = spacy.load("en_core_web_md")

# 2つのdocを比較
doc1 = nlp("I like fast food")
doc2 = nlp("I like pizza")
print(doc1.similarity(doc2))
```

```out
0.8627204117787385
```

```python
# 2つのトークンを比較
doc = nlp("I like pizza and pasta")
token1 = doc[2]
token2 = doc[4]
print(token1.similarity(token2))
```

```out
0.7369546
```

Notes: 例を見てみます。2つのdocが似ているかどうかを調べたいとしましょう。

まずはじめに、中サイズの英語モデル「en_core_web_md」をロードしましょう。

次に、2つのdocオブジェクトを作れば、1つめのdocオブジェクトの`similarity`メソッドを2つめのdocオブジェクトに適用し、
類似度を算出することができます。


ここでは、「ファーストフードが好き」と「ピザが好き」の類似度スコアが0.86となり、かなり高いことが予測されています。

同じ方法をトークンにも用いることができます。

単語ベクトルによると、「ピザ」と「パスタ」のトークンは似たようなもので、スコアは0.7点となっています。

---

# 類似度の例(2)

```python
# docとtokenを比較
doc = nlp("I like pizza")
token = nlp("soap")[0]

print(doc.similarity(token))
```

```out
0.32531983166759537
```

```python
# spanとdocを比較
span = nlp("I like pizza and pasta")[2:5]
doc = nlp("McDonalds sells burgers")

print(span.similarity(doc))
```

```out
0.619909235817623
```

Notes: `similarity`メソッドは、異なるオブジェクトに対しても用いることができます。

例えば、docとtokenに対して使えます。

ここでは、類似度はかなり低く、2つのオブジェクトはあまり似ていないと考えられます。

この例は、「pizza and pasta」というスパンと、マクドナルドに関するdocを比較しています。

類似度は0.61となっており、似たようなものであると考えられます。

---

# 類似度の算出法

- 類似度は**単語ベクトル**を用いて決められています
- 単語の意味を多次元で表しています
- [Word2Vec](https://en.wikipedia.org/wiki/Word2vec)のようなアルゴリズムと、大量のテキストを用いて生成されます
- spaCyの機械学習モデルに組み込むことができます
- デフォルトでコサイン類似度が用いられますが、変更可能です
- `Doc`と`Span`ベクトルは、デフォルトではトークンのベクトルの平均です
- 長くて無関係の単語がたくさん含まれている文章よりも、短いフレーズの方が効果的です

Notes: さて、spaCyは裏側で何をしているのでしょうか？

類似度は、単語の意味を表現する多次元の単語ベクトルを用いて決定されます。

Word2Vecというアルゴリズムを聞いたことがあるかもしれませんが、これは生のテキストから単語ベクトルを学習するのによく使われるアルゴリズムです。

単語ベクトルはspaCyの機械学習モデルに組み込むことができます。

デフォルトでは、類似度は2つのベクトルのコサイン類似度を用いて計算されますが、必要に応じて変更可能です。

`Doc` や `Span` のように複数のトークンからなるオブジェクトのベクトルは、それらのトークンベクトルの平均値がデフォルトとなります。

なので通常、無関係な単語が少ない短いフレーズの方が有用です。

---

# spaCyの単語ベクトル

```python
# 単語ベクトルの入っている大きなモデルをロード
nlp = spacy.load("en_core_web_md")

doc = nlp("I have a banana")
# token.vector属性によって単語ベクトルを取得
print(doc[3].vector)
```

```out
 [2.02280000e-01,  -7.66180009e-02,   3.70319992e-01,
  3.28450017e-02,  -4.19569999e-01,   7.20689967e-02,
 -3.74760002e-01,   5.74599989e-02,  -1.24009997e-02,
  5.29489994e-01,  -5.23800015e-01,  -1.97710007e-01,
 -3.41470003e-01,   5.33169985e-01,  -2.53309999e-02,
  1.73800007e-01,   1.67720005e-01,   8.39839995e-01,
  5.51070012e-02,   1.05470002e-01,   3.78719985e-01,
  2.42750004e-01,   1.47449998e-02,   5.59509993e-01,
  1.25210002e-01,  -6.75960004e-01,   3.58420014e-01,
 -4.00279984e-02,   9.59490016e-02,  -5.06900012e-01,
 -8.53179991e-02,   1.79800004e-01,   3.38669986e-01,
  ...
```

Notes: これらのベクトルがどのようなものであるかのイメージを掴むための例を示しています。

最初に、単語ベクトルが入っている中サイズのモデルをロードします。

次に、テキストを処理し、`.vector`属性によってトークンのベクトルを取得します。

結果として、「banana」を表す300次元のベクトルが出力されます。

---

# 類似度はアプリケーションに依存

- 推薦システム、重複検出等、様々なアプリケーションに用いることができます
- 「類似度」の客観的な定義はありません
- アプリケーションのコンテキストや、目的に依存します

```python
doc1 = nlp("I like cats")
doc2 = nlp("I hate cats")

print(doc1.similarity(doc2))
```

```out
0.9501447503553421
```

Notes: 類似度の予測は、多くのアプリケーションに用いることができます。
例えば、ユーザに対して、今まで読んだ文章をもとに似た文章を推薦するシステムなどです。
また、オンラインプラットフォーム上の投稿のように、重複するコンテンツの検出にも役立ちます。

しかし、何が似ていて何が似ていないかという客観的な定義は存在しないことを心に留めておいてください。
これはコンテキストと、アプリケーションの目的に依存します。

これが一例です。spaCyのデフォルトの単語ベクトルは、「I like cats」と「I hate cats」の類似度が非常に高いと予測します。
これらの文は両方とも、猫に関する感情について表しているので、結果は理にかなっています。
しかし、これらの文は正反対の感情を述べているので、別のアプリケーションで用いる際は「全く似ていない」という予測結果が欲しくなるかもしれません。

---

# Let's practice!

Notes: それでは手を動かしていきましょう。実際にspaCyの単語ベクトルを使ってみて類似度の予測をしてみましょう。
