REASONING AGENT
===============
Decides whether a query follows from a knowledge base written in first-order logic.
Answer: True (the query is entailed) or False (it is not).
Method: formulas are turned into clauses, and resolution refutation with unification searches for a
contradiction between the knowledge base and the negated query. One engine is used for every item.
No machine learning and no generative AI is used.


FILES
-----
reasoning_agent.ipynb   the complete system; prints accuracy and inference time and writes the predictions
submission.csv          predictions (id, flag, label) on the public data used (3028 examples)
report.pdf              the method, the datasets and the results
README.txt              this file


REQUIREMENTS
------------
Python 3.9 or newer (the reported results were run with Python 3.14):   pip install pandas jupyter
Only to reproduce the reported results:   pip install datasets


HOW TO RUN
----------
1. Put the test file next to the notebook as test.csv (.tsv, .json, .jsonl, .parquet also work),
   or set TEST_FILE = "name_of_test_file" in the first code cell.
2. Run all cells. The last cells print the number of examples, the total inference time, the accuracy
   (if the file has labels) and the TP / FN / FP / TN counts, and write submission.csv.
   Only the reasoning is timed.
Without a test file the notebook runs the public-data experiments (RuleTaker, CounterBench, FOLIO).


INPUT FORMAT
------------
Columns (found by name):
    id          example id
    context     the knowledge base, one formula per line   (also: premises, premises-FOL, kb)
    statement   the query, one formula                     (also: conclusion, conclusion-FOL, query)
    label       optional: True / False (anything else, e.g. Uncertain, counts as False)   (also: flag)

Example:
    context:    Young(Bob)
                Kind(Bob)
                ∀x (Young(x) → Smart(x))
                ∀x (Smart(x) ∧ Kind(x) → Successful(x))
    statement:  Successful(Bob)          ->  True

Output: submission.csv with the columns id, flag, label (flag and label are the same answer).


LOGIC SYNTAX
------------
    not  ¬ ~        and  ∧ &        or  ∨ |        exclusive or  ⊕ xor
    implies  → ->   if and only if  ↔ <->           equality  = ≠
    for all  ∀x     there exists  ∃x
Predicates are written Name(arg1, arg2); a single name such as rain is a proposition.
Unquantified x, y, z, w are universal variables. Names are not case-sensitive.


SETTINGS (first code cell)
--------------------------
    TEST_FILE               test file path (None = look for test.* next to the notebook)
    LABEL_STYLE             True/False (default), 1/0 or yes/no in the output
    INCONSISTENT_KB_ANSWER  answer for a knowledge base that contradicts itself (default False)
    ACCEPT_ENGLISH          False (default; the input is logic)


RESULTS ON THE PUBLIC DATA (details in report.pdf)
--------------------------------------------------
    Dataset                                   Examples   Accuracy   Time (s)
    RuleTaker (jise, all splits)                  1625     98.46%       0.38
    CounterBench                                  1200     95.00%       0.59
    FOLIO validation (Uncertain = False)           203     86.21%       3.83
    All                                           3028     96.27%       4.80

Speed-ups: search from the negated query, shortest clauses first, predicate index, simplification
with known facts, removal of useless clauses, and a limit of 0.25 s per item (then the answer is False).


ASSUMPTIONS
-----------
    - The test input is logic; output labels are True / False.
    - FOLIO: True -> True; False and Uncertain -> False. RuleTaker: entailment -> True. CounterBench: yes -> True.
    - RuleTaker and CounterBench are English; for the experiments they are written out as logic by
      small converters in the notebook. The agent itself reads only logic.
    - The private test set is not available, so submission.csv holds the predictions on the public data
      (ids like ruletaker-12, counterbench-7, folio-validation-3).


REPRODUCING THE REPORTED RESULTS
--------------------------------
1. pip install datasets
2. FOLIO needs a Hugging Face login (once): accept the terms at
   https://huggingface.co/datasets/yale-nlp/FOLIO, create a "Read" token at
   https://huggingface.co/settings/tokens, then run  huggingface-cli login  and paste it.
   RuleTaker and CounterBench need no login.
3. Leave TEST_FILE = None, make sure there is no test.* file next to the notebook, and run all cells.


DATASETS
--------
RuleTaker     https://huggingface.co/datasets/jise/ruletaker
CounterBench  https://huggingface.co/datasets/CounterBench/CounterBench
FOLIO         https://huggingface.co/datasets/yale-nlp/FOLIO
