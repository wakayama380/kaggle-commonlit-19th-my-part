<h1>19th-Place-Solution<h1>
<h2>Final Model</h2>
<p>For the final submission, different features and weights were used:</p>
<ul>
  <li>Sub1 (TrustLB) LB:0.422 / CV:0.463-0.465 (using a different CV calculation method)</li>
  <li>Sub2 (TrustCV) LB:0.448 / CV:0.458</li>
  <li>Sub3 (Balance) LB:0.441 / CV:0.460</li>
</ul>
<h2>Transformer</h2>
<ul>
  <li>Frozen layers</li>
  <li>Concatenation of deberta-v3-large embeddings to the final layer before training (only for model-p1)</li>
  <li>Prediction using CLS tokens</li>
  <li>Linear scheduler with warm-up, 3 epochs</li>
</ul>
<ul>
  <li>Frozen layers</li>
  <li>Concat pooling (concatenating cls tokens from multiple layers)</li>
</ul>
<h2>LightGBM</h2>
<p>Ultimately, about 80 features were used. For each Transformer model, 10 seeds x 4 folds of LightGBM Average were calculated. When adding features, improvements in CV from a 50-seed average were checked.</p>
<h2>Features</h2>
<p>Features from public notebooks:</p>
<ul>
  <li>Jaccard coefficient between question/prompt_text and summary_text.</li>
  <li>Calculate tfidf for summary_text per prompt, and take the average tfidf per record.</li>
  <li>Include only words not in prompt_text.</li>
  <li>Take the average of all columns and the average of non-zero elements.</li>
  <li>Cosine similarity of average tfidf/BoW of summary_text per prompt.</li>
  <li>Sentence Transformer embedding features.</li>
  <li>Cosine similarity between prompt_text and summary_text.</li>
  <li>Cosine similarity between question and summary_text.</li>
  <li>Cosine similarity between average embedding of summary_text per prompt.</li>
  <li>Calculate cosine similarity between embeddings of sentences split from prompt_text and summary_text, and take the standard deviation of similarity for all sentences.</li>
  <li>Similarly, calculate cosine similarity between embeddings of sentences split from summary_text and prompt_text, and take the standard deviation of similarity for all sentences.</li>
  <li>For each prompt, perform kNN with the embedding of summary_text.</li>
  <li>Extract features and cosine similarity of the top 5% of oof features with high similarity for each record.</li>
  <li>Take the average of BERT scores and embedding features by extracting text from the beginning, middle, and end for long prompts.</li>
  <li>Include prediction values from Feedback competition 3 models as features.</li>
  <li>Features from pyreadability.</li>
  <li>Take the average general frequency of words present only in summary_text and not in prompt_text for each summary_text.</li>
</ul>
<h2>Ensemble</h2>
<p>Weight optimization was done using Nelder-Mead. Only weights above 0 were used, and they were rounded to a total of 1.0. Ensemble weights were changed according to the token count of summary_text + prompt_text. For token counts above a certain threshold, the weight of raw Transformer predictions was gradually reduced, and the weight of LightGBM was increased (linearly adjusted by token count). It was assumed that when the token count is high, the prompt_text does not fit entirely in the Transformer with prompt_text input, reducing accuracy.</p>


![image](https://github.com/wakayama380/kaggle-commonlit-19th-my-part/assets/96282801/ef35ccdf-838f-4189-a6d7-e8cafafbce55)
