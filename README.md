<h2>最終モデル</h2>

<p>最終提出用に、異なる特徴と重みを</p>
<ul>
  <li>Sub1 (TrustLB) LB:0.422 / CV:0.463-0.465 (異なるCV計算方法)</li>
  <li>Sub2 (TrustCV) LB:0.448 / CV:0.458</li>
  <li>Sub3 (Balance) LB:0.441 / CV:0.460</li>
</ul>

<h2>Transformer</h2>

<h3>pao モデル</h3>
<ul>
  <li>凍結レイヤー</li>
  <li>訓練前に deberta-v3-large の埋め込みを最終層に連結 (model-p1 のみ)</li>
  <li>CLS トークンを使用した予測</li>
  <li>ウォームアップ付きのリニアスケジューラ、3エポック</li>
</ul>

<h3>takai モデル</h3>
<ul>
  <li>凍結レイヤー</li>
  <li>concat pooling (複数のレイヤーの cls トークンを連結)</li>
</ul>

<h2>LightGBM</h2>

<p>最終的には約80の特徴量があります。各Transformerモデルについて10シード x 4 folds のLightGBM Averageが行われます。特徴を追加する際は、50シードの平均スコアからCVが改善されたかを確認します。</p>

<h2>特徴</h2>

<p>公開ノートブックからの特徴:</p>
<ul>
  <li>質問/prompt_text と summary_text の Jaccard 係数。</li>
  <li>promptごとにsummary_textのtfidfを計算し、レコードごとの平均tfidfを取得。</li>
  <li>全てのsummary_textとprompt_textに含まれない単語のみ。</li>
  <li>全列の平均と非ゼロ要素の平均を取る。</li>
  <li>promptごとのsummary_textの平均tfidf/BoWのcosine similarity。</li>
  <li>Sentence Transformer埋め込み特徴量。</li>
  <li>prompt_textとsummary_textのcosine similarity。</li>
  <li>質問とsummary_textのcosine similarity。</li>
  <li>promptごとのsummary_textの平均埋め込みのcosine similarity。</li>
  <li>prompt_textから分割された文の埋め込みとsummary_textの間のcosine similarityを計算し、すべての文の類似度の標準偏差を取得。</li>
  <li>同様に、summary_textから分割された文の埋め込みとprompt_textの間のcosine similarityを計算し、すべての文の類似度の標準偏差を取得。</li>
  <li>promptごとに、summary_textの埋め込みでkNNを行います。</li>
  <li>各レコードに対して高い類似度を持つトップ5%のoof特徴量の平均とcosine similarityを抽出します。</li>
  <li>全てのBERTスコアと埋め込み特徴量について、長いプロンプトからテキストを抽出して平均を取ります。</li>
  <li>フィードバックコンペティション3のモデルからの予測値を特徴量として含みます。</li>
  <li>pyreadabilityからの特徴量。</li>
  <li>summary_textにのみ存在し、prompt_textに存在しない単語の一般頻度の平均。</li>
</ul>

<h2>アンサンブル</h2>

<p>Nelder-Meadを使用した重み最適化。最終的には0より大きい重みのみを使用し、合計が1.0になるように丸めます。summary_text + prompt_textのトークン数に応じてアンサンブルの重みを変更します。特定のトークン数以上の場合は、生のTransformerの予測値の重みを減らし、LightGBMの重みを増やします（トークン数に応じて重みを線形に調整）。トークン数が高い場合、prompt_textが完全にTransformerに適合せず、精度が低下すると仮定されています。</p>
