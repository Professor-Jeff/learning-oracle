# The Learning Oracle — A Magic 8-Ball You Can Teach

**Live (try it in the browser):** https://professor-jeff.github.io/learning-oracle/

Fourth in the COMP 138 series after [pick-activity](https://github.com/Professor-Jeff/pick-activity),
[tictactoe-nn](https://github.com/Professor-Jeff/tictactoe-nn) and
[happy-face-nn](https://github.com/Professor-Jeff/happy-face-nn). The first three learned from
examples the student handed them. This one talks back, one word at a time, and learns from how the
student rates what it says: the idea behind training a chatbot on human feedback, shrunk to a
25-word vocabulary. It is the bridge to ChatGPT.

## The demo script

1. **Classic 8-Ball.** Twenty stock answers, `Math.random()`, no model. Nothing about the question
   reaches the answer and no rating can change it, so there are no thumbs in this mode.
2. **Switch to Learning Oracle.** Same ball, same question box. A real neural net now answers,
   trained in the browser on page load (200 epochs, under a second, loss curve shown). Every word
   choice is shown with the probabilities the net gave each candidate.
3. **Ask 10×.** Same question, ten shakes, tallied. Thumbs-up or -down the last answer, press
   Ask 10× again, and the spread tilts. One thumbs up typically turns a 5/5 split into 10/0.
4. **Curate.** Starter question/answer pairs shuffled with word-salad decoys; approve or reject each,
   then *Retrain on approved*. The Oracle learns only from what was approved.
5. **Vocabulary and corpus.** Replace the word list and write a new `question => answer` corpus.
   Lines with out-of-vocabulary words are flagged and dropped: vocabulary constrains what a model
   can say.

## The model (`class Oracle` in `index.html`)

Plain JavaScript, no libraries. A tiny next-word predictor:

- **Input**, 75 numbers for a 25-word vocabulary: which vocab words appear in the question (25),
  plus one-hot of the last word said (26, with START) and the word before that (26).
- **Hidden**: 24 tanh neurons. **Output**: softmax over the 25 words plus END. 2,522 weights.
- **Generation**: start from (START, START), sample a word at temperature 0.8, feed it back, repeat
  until END or 8 words. The top five candidates at each step are recorded for the "what it was
  thinking" panel.
- **Training**: each pair becomes one example per answer word plus one for END. Plain SGD,
  cross-entropy, learning rate 0.08, shuffled each epoch. Seeded RNG so every load trains the
  same model.
- **Thumbs up**: six extra SGD steps on the exact (question, answer) pair. **Thumbs down**: six
  steps of an unlikelihood loss, `-log(1 - p)`, with the step clipped so it cannot blow up.
  Feedback reports the total |Δweight| so students see the weights move.

No persistence: the net retrains from seed on every load and thumbs vanish on refresh.

## Deploying

Edit `index.html` and push to `main`. GitHub Pages redeploys within about a minute.
