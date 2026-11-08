# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：
回答内容⇒AIE2026の宿題における「3レベル」とは、宿題の難易度や内容の構成を示す以下の3段階のことです。

1. **基礎**：講義内容の確認を行うレベル
2. **実装**：自分でコードを書くレベル
3. **発展**：より高度な調査や試行を行うレベル

3つのうち**どれか1つに取り組めばその回の宿題は完了**となります（原則として採点は行われません）。ご自身の理解度や目的に合わせて選んで取り組んでみてくださいね。

第1回

比較結果⇒素のLLMとは違い、正解のjsonの内容を回答した。
- `--trace` で見えたツール呼び出し（ツール名と引数）：
回答内容⇒講座の用語集によると、LoRAとQLoRAの違いは以下のようになります。

* **LoRA**: 事前学習済みの重みは固定したまま、重みの更新分を2つの小さな低ランク行列の積で表し、その部分だけを学習するPEFT手法です。学習パラメータが少なく、アダプタのファイルサイズも小さくなります。
* **QLoRA**: ベースモデルの重みを4ビットなどに**量子化**してメモリを大幅に節約したうえで、LoRAによる学習を行う手法です。これにより、さらに大きなモデルを小さなGPU環境でも扱えるようになります。

第2回

（関連する回：第2回）

--- trace ---
[tool call]   search_glossary({'query': 'LoRA'})
[tool call]   search_glossary({'query': 'QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
[usage] RunUsage(cost=Decimal('0.0008169'), details={'text_prompt_tokens': 1198}, input_text_tokens=1198, input_tokens=1198, output_tokens=183, requests=2, tool_calls=2)

ツール呼び出しの結果⇒
①ツール：search_glossary、引数：{'query': 'LoRA'}
②ツール：search_glossary、引数：{'query': 'QLoRA'}

- 1つの質問での LLM 呼び出し回数（`requests`）：2回

- 追加したツールと、docstring に書いた説明：
def make_quiz(lecture: int) -> str:
    """指定した回の用語から1つを選び、確認問題と答えをJSONで返す。

    「第2回の確認問題を出して」のように、
    復習用の問題を求められたときに使う。
    """
    terms = [entry for entry in load_glossary() if entry["lecture"] == lecture]
    if not terms:
        return json.dumps(
            {"error": f"第{lecture}回の用語は用語集にありません。"},
            ensure_ascii=False,
        )

    term = random.choice(terms)
    return json.dumps(
        {
            "question": f"{term['term']}の意味は？",
            "answer": term["definition"],
        },
        ensure_ascii=False,
    )


## 宿題

- 取り組んだレベル：基礎
- やったこと：search_glossaryのdocstring の「用語を1つだけ入れる」を「まとめて入れる」に変えて、質問「uv run python -m app.agent --trace "LoRA とQLoRAの違いは？ "」を行うと、検
索の回数がどう変わるか
- 結果：
【①用語を1つだけ入れるの場合の回答】検索回数：2回
uv run python -m app.agent --trace "LoRA とQLoRAの違いは？ "
[model] gemini-3.5-flash-lite
LoRAとQLoRAの違いは、ベースモデルの重みをそのまま使うか、量子化してメモリを削減するかにあります。

* **LoRA（Low-Rank Adaptation）**
  事前学習済みの重みは固定したまま、重みの更新分を低ランク行列（2つの小さな行列の積）で近似し、その小さな行列だけを学習するPEFT手法です。学習するパラメータ数が非常に少なく、効率よくファインチューニングを行えます。
* **QLoRA（Quantized LoRA）**
  LoRAの仕組みに加え、ベースモデルの重みを4ビットなどの低ビットに**量子化**してメモリ消費を大幅に抑えた状態で学習する手法です。これにより、より大きなモデルを比較的小さなGPUメモリでも扱えるようになります。

関係する回：第2回

--- trace ---
[tool call]   search_glossary({'query': 'LoRA'})
[tool call]   search_glossary({'query': 'QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool return] search_glossary -> [{"term": "QLoRA", "lecture": 2, "definition": "ベースモデルの重みを4ビットなどに量子化してメモリを節約したうえで、LoRAで学習する手法。より大きなモデルを小さなGPUで扱える。本講座の第2…
[usage] RunUsage(cost=Decimal('0.0009562'), details={'text_prompt_tokens': 1404}, input_text_tokens=1404, input_tokens=1404, output_tokens=214, requests=2, tool_calls=2)

【②まとめて入れるの場合の回答】検索回数：1回
uv run python -m app.agent --trace "LoRA とQLoRAの違いは？ "
[model] gemini-3.5-flash-lite
LoRA（Low-Rank Adaptation）とQLoRA（Quantized LoRA）の違いは、主に**ベースモデルの重みの保持方法と精度**にあります。

* **LoRA**
  * **特徴:** 事前学習済みの重みを元の精度のまま（通常はFP16やBF16など）固定し、重みの更新分を低ランク行列（小さな行列）で近似して、その部分だけを学習するPEFT（Parameter-Efficient Fine-Tuning）手法です。
  * **メリット:** 学習パラメータ数が大幅に削減され、効率的にファインチューニングができます。

* **QLoRA**
  * **特徴:** LoRAの手法をベースにしつつ、ベースモデルの重みを**4ビットなどに量子化**（圧縮）してメモリ消費を極限まで抑えたうえで、LoRAによる学習を行います。
  * **メリット:** メモリ使用量が劇的に減るため、ローカルの小さなGPU環境でも、より大規模なモデルのファインチューニングが可能になります。

関連する回：第2回

--- trace ---
[tool call]   search_glossary({'query': 'LoRA QLoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[usage] RunUsage(cost=Decimal('0.0009604'), details={'text_prompt_tokens': 1093}, input_text_tokens=1093, input_tokens=1093, output_tokens=253, requests=2, tool_calls=1)

- 気づき・疑問：まとめて検索することで、ツール使用回数が減った。input_tokensも検索回数が低いほうが低くなったが、output_tokensは検索回数が少ないほうが多く、cost=Decimalも検索回数が少ないほうがやや多かった。INSTRUCTIONSやdocstringでエージェントやツールのふるまいを調節できることがわかった。
