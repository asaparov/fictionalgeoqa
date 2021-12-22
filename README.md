# FictionalGeoQA

A question-answering dataset containing paragraphs and questions about fictional geography, designed to test reasoning ability. If you use this data or code in your research, please cite:

TODO: Update this to MIT Press version once published.
```bibtex
@article{SaparovMitchell2021,
  author = {Abulhair Saparov and Tom M. Mitchell},
  title = {Towards General Natural Language Understanding with Probabilistic Worldbuilding},
  journal = {Transactions of the Association for Computational Linguistics (TACL)},
  pages = {(accepted)},
  year = {2021}
}
```


## Data

The data is located in [`fictionalgeoqa.jsonl`](fictionalgeoqa.jsonl).

Each line contains one example: a paragraph context (the JSON field `theory`), a single question (the JSON field `question`), and an answer (the JSON field `answer`). For examples that have multiple answers, the answer field will be a comma-separated list of the answers. For examples with no answers, the answer field is an empty string.

To accommodate language model-based QA systems that may overgenerate text, each example has an `answer_templates` field, which is a list of regular expressions. A predicted answer is considered correct either if it matches the true answer, or it matches any regular expression answer_template, where "{0}" is replaced with the true answer. Case is ignored when judging correctness. For example, consider [line 11](fictionalgeoqa.jsonl#L11):
```json
"question":"What are the major rivers in Wulstershire?","answer":"River Giffeleney, River Wulstershire" ...
"answer_templates":["{0} is a river in Wulstershire","{0} is a major river( in Wulstershire)?","{0} is major"]
```
The answer "river Giffeleney, River wulstershire is major" would be considered correct, since the first predicted answer "river Giffeleney" matches the first true answer exactly (ignoring case), and the second predicted answer "River wulstershire is Major" matches the second true answer when substituted for "{0}" in the answer template "{0} is major".

All true answers must be predicted to be considered correct; no more, no less. **_Please report an issue_** if you think a predicted answer should be considered correct but is not.

The Python script [`check_fictionalgeoqa_answers.py`](check_fictionalgeoqa_answers.py) performs this evaluation automatically (see [below](#evaluation)).

The `data_subsets` field is a list of flags that indicate the types of reasoning that are required to correctly answer the question. The possible flags are:
 - `superlative`: Examples that require reasoning over superlatives, i.e. "longest river."
 - `subjective_concept_definition`: Examples with definitions of "*subjective*" concepts, i.e. ``Every river longer than 500 km is **major**.''
 - `objective_concept_definition`: Examples with definitions of "*objective*" concepts, i.e. the **population** of a location is the number of people living there.
 - `lexical_ambiguity`: Examples with lexical ambiguity, i.e. "has" means different things in "a state has a city named" vs "a state has an area of..." The phrase "largest state" could either mean "state with the largest area" or "state with the largest population."
 - `negation`: Examples that require reasoning with classical negation (negation-as-failure is insufficient).
 - `large_context`: Examples where there are at least 100 sentences in the context (JSON field `theory`).
 - `arithmetic`: Examples that require simple arithmetic.
 - `counting`: Examples that require counting.


## Evaluation

Given a text file containing a list of predicted answers, one for each line, the Python script [`check_fictionalgeoqa_answers.py`](check_fictionalgeoqa_answers.py) automatically checks the correctness of the predictions (and breaks down the results across data_subsets).
```bash
python check_fictionalgeoqa_answers.py [predicted answers filepath]
```

**_Please report an issue_** if you think a predicted answer should be considered correct but is not.


## Experiments

### PWL

Please see the repository [PWL](https://github.com/asaparov/PWL).

### UnifiedQA

First install [`transformers`](https://github.com/huggingface/transformers), then simply run
```bash
python run_unifiedqa.py > [predicted answers filepath]
```

### Boxer

First install [C&C/Boxer](https://github.com/chbrown/candc) and either [E prover](https://github.com/eprover/eprover) or [Vampire](https://github.com/vprover/vampire).

To run using the E prover:
```bash
python run_boxer.py [path to C&C/Boxer directory] [path to E prover directory] [predicted answers filepath]
```
The E prover directory must contain a folder `PROVER` with the executable `eprover` within.

To run using Vampire:
```bash
python run_boxer.py [path to C&C/Boxer directory] [path to Vampire directory] [predicted answers filepath] --use-vampire --conjectures
```

### NeuralDRS

First install the [Marian-based Neural DRS parser](https://github.com/RikVN/Neural_DRS/blob/master/Marian.md) and either [E prover](https://github.com/eprover/eprover) or [Vampire](https://github.com/vprover/vampire).

To run using the E prover:
```bash
python run_neural_drs.py [Neural DRS directory] [path to E prover directory] [predicted answers filepath]
```
The E prover directory must contain a folder `PROVER` with the executable `eprover` within.

To run using Vampire:
```bash
python run_neural_drs.py [Neural DRS directory] [path to Vampire directory] [predicted answers filepath] --use-vampire --conjectures
```

### PWL language module + E/Vampire

The file [`fictionalgeoqa_parse_outputs.txt`](fictionalgeoqa_parse_outputs.txt) contains the output of the language module of [PWL](https://github.com/asaparov/PWL) for every paragraph and question in the dataset (one line for each example). The script [`run_thm_prover.py`](run_thm_prover.py) reads this file and feeds the logical forms into the first-order theorem provers to predict answers.

First install either [E prover](https://github.com/eprover/eprover) or [Vampire](https://github.com/vprover/vampire).

To run using the E prover:
```bash
python run_thm_prover.py [path to E prover directory] [predicted answers filepath]
```
The E prover directory must contain a folder `PROVER` with the executable `eprover` within.

To run using Vampire:
```bash
python run_thm_prover.py [path to Vampire directory] [predicted answers filepath] --use-vampire --conjectures
```
