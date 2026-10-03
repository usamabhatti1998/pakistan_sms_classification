# Pakistan SMS classification dataset

Exploratory analysis of a small labelled SMS dataset from Kaggle:
https://www.kaggle.com/datasets/muhammadhumyun/pakistan-sms-classification-dataset

The file `Pakistan_sms_dataset.csv` has 4,895 rows and 3 columns: `Label` (Normal, Promotional, fraud),
`Messages` (the raw text) and `clean_message` (a lowercased, punctuation-stripped copy made by the uploader).
There are no dates, senders or other fields, and no data dictionary.

## Headline numbers

- 4,895 raw rows, 4,884 after cleaning (2 chatbot leftover rows, 4 conflicting-label rows and 5 duplicates removed)
- Normal 2,501 (51.2%), Promotional 1,276 (26.1%), Fraud 1,107 (22.7%)
- Median length: Normal 71 characters, Fraud 144, Promotional 155
- English only: no Urdu script, effectively no Roman Urdu
- 40.4% of fraud messages are close rewordings of another fraud message (6.8% of Normal, 4.8% of Promotional)
- TF-IDF + logistic regression baseline: 0.983 accuracy, 0.978 macro F1 with near-duplicates kept on one side
  of the split (0.991 / 0.988 on a plain random split)

## Is the data real?

Only partly. It is a blend of three things: old-style English spam with UK premium numbers and pound prices
(it reads like a legacy SMS spam corpus, though the file does not say so), Pakistan-specific scam and promo
messages, and a large amount of LLM-written paraphrase. Two rows in the raw file are leftover chatbot replies
("Certainly, here are 50 unique variations of the message..."), which confirms the last part.

## Other findings

- Longest message: 582 characters (Promotional). Shortest: 12 characters (Normal).
- 43.5% of promotional and 27.0% of fraud messages exceed the 160-character single-SMS limit; 0.3% of normal ones do.
- In the raw file 91.2% of Normal and 70.8% of Promotional messages are wrapped in literal quote marks, against
  3.5% of fraud. Quote mark plus length alone gives 0.81 accuracy (majority baseline 0.51).
- The Benazir Income Support Programme is named in 48.1% of fraud messages (527 messages), the most mentioned
  name in the dataset. Jeeto Pakistan and other TV lucky draws: 17.7%. Bank / ATM / card: 15.5%.
- Prize, win or cash wording: 72.6% of fraud messages. Money amount: 55.1%. Phone number: 50.0%. WhatsApp: 22.4%.
- Promotional messages: 39.7% discount or offer wording, 20.1% hashtags, 18.4% links, 17.0% short codes or USSD.
- No normal message contains a phone number, link or hashtag.
- 483 distinct phone numbers appear in 742 messages. One number is shared by 51 fraud messages and another by 49;
  62 of the distinct numbers contain placeholder-style running digits.
- Largest near-duplicate group: 13 messages, all rewordings of the same Benazir cash-prize message.
- Largest plain rupee figure quoted in a fraud message: 84,500 (median 23,500 over 457 messages).
- Of 17 baseline errors on 977 test messages, 12 are Fraud/Promotional mix-ups, mostly UK-style spam that is
  labelled inconsistently between those two classes.

## Caveats

- Not a sample of real Pakistani SMS traffic. Provenance is undocumented; the source judgements come from reading
  the text and from artefacts in it.
- The three classes were assembled differently (quoting, templating, origin), so a model can separate them on
  style as much as on meaning. Classifier scores describe this file only.
- The raw file is sorted by label, so shuffle before modelling.
- Near-duplicate grouping uses a 0.8 cosine cut-off and misses looser rewordings.
- Pattern flags (phone, money, urgency and so on) are regular expressions and are approximate.
- Phone numbers are masked in the notebook output and the report. They are still present in the original file
  and in `sms_cleaned.csv`, since removing them would change the text.

## Folder layout

    Pakistan_sms_dataset.csv   original data, untouched
    sms_cleaned.csv            cleaned messages plus length, pattern and near-duplicate columns
    sms_analysis.ipynb         the analysis, saved with outputs
    sms_report.pdf             written report (5 pages)
    README.md
    charts/                    PNG charts produced by the notebook

## How to run

From this folder, with the shared environment one level up:

    ../.venv/bin/python -c "import nbformat; from nbclient import NotebookClient; nb = nbformat.read('sms_analysis.ipynb', 4); NotebookClient(nb, kernel_name='python3').execute(); nbformat.write(nb, 'sms_analysis.ipynb')"

or open `sms_analysis.ipynb` in Jupyter and run all cells. It needs pandas, numpy, matplotlib, scipy and
scikit-learn, uses relative paths, and rewrites `sms_cleaned.csv` and the files in `charts/`.
