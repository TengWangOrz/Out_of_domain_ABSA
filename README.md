# Out_of_domain_ABSA

This repo will do "out of domain" aspect based sentiment analysis. It means that you will train a ABSA(Aspect bsaed sentiment analysis) model and use another dataset which is not relevant dataset, get the aspect and sentiment.

You can run the [notebook](https://colab.research.google.com/drive/1LfNqhqheVeY8YrsBI1l3IVlNQp_J6pl5#scrollTo=efxtO9cYNHk6)

At first, you need to create a aspect detection model. I just use the [SpanEmo](https://github.com/hasanhuz/SpanEmo), this SpanEmo will get all of the aspects of sentence. Then you need to train the SpanEmo and build a model to output the aspect_polarity.

First of all, let's build the sentiment detection model. You need to load your data in "data" folder. You can see the data format with my preprocessing data in the 'data' folder.

Then you can train the model like this.
```
!python scripts/train.py    --train-path {"data/sentihood-train.tsv"}\
                            --dev-path {"data/sentihood-dev.tsv"} \
                            --bert-type {"base-bert"}\
                            --max-length 128 \
                            --output-dropout 0.1 \
                            --seed 0 \
                            --train-batch-size 32 \
                            --eval-batch-size 32 \
                            --max-epoch 20 \
                            --ffn-lr 0.001 \
                            --bert-lr 2e-5 

```
the detail of parameters are as follows:
```
"""
Usage:
    main.py [options]

Options:
    -h --help                         show this screen
    --max-length=<int>                text length [default: 128]
    --output-dropout=<float>          prob of dropout applied to the output layer [default: 0.1]
    --seed=<int>                      fixed random seed number [default: 42]
    --train-batch-size=<int>          batch size [default: 32]
    --eval-batch-size=<int>           batch size [default: 32]
    --max-epoch=<int>                 max epoch [default: 20]
    --ffn-lr=<float>                  ffn learning rate [default: 0.001]
    --bert-lr=<float>                 bert learning rate [default: 2e-5]
    --bert-type=<str>                 language choice [default: base-bert]
    --dev-path=<str>                  file path of the dev set [default: '']
    --train-path=<str>                file path of the train set [default: '']
"""
```
After having already trained the model. We can validation it with our test dataset. Like this
 
```
!python scripts/test.py --test-path {'data/ABSA_15_Restaurants_Test.tsv'} \
                        --model-path {"2021-11-12-16:39:24_checkpoint.pt"} \
                        --max-length 160 \
                        --bert-type {"base-bert"}
```

the detailed of parameters are as follows:
```
"""
Usage:
    main.py [options]

Options:
    -h --help                         show this screen
    --models-path=<str>                path of the trained models
    --max-length=<int>                text length [default: 128]
    --seed=<int>                      seed [default: 0]
    --test-batch-size=<int>           batch size [default: 32]
    --bert-type=<str>                      language choice [default: base-bert]
    --test-path=<str>                 file path of the test set [default: ]
"""
```
Next, you need to train SpanEmo.

```
%cd SpanEmo
!python scripts/train.py    --train-path {"SemEval14/aspect_restaurants_train.csv"}\
                            --dev-path {"SemEval14/aspect_resuaurants_dev.csv"} \
                            --loss-type {'cross-entropy'} \
                            --max-length 128 \
                            --output-dropout 0.1 \
                            --seed 42 \
                            --train-batch-size 32 \
                            --eval-batch-size 32 \
                            --max-epoch 20 \
                            --ffn-lr 0.001 \
                            --bert-lr 2e-5 \
                            --lang {"English"} \
                            --bert-type {'BERT'}
```

After training, you will get the model checkpoint, you need to load it when validation the model. 

```
!python scripts/test.py --test-path {'SemEval14/test_data.csv'} \
                        --model-path {"/content/Out_of_domain_ABSA/SpanEmo/models/2021-11-13-11:58:42_checkpoint.pt"}
```

Then, you have got the aspects of sentence in predict.csv. You need to run this code, to get the right format of data that before feeding sentiment detection.
```
%cd scripts
!python data_integration.py
%cd ../

%cd SemEval14
!python data_preprocess_for_SA.py
%cd ../

%cd ../scripts
!python faker_data.py
%cd ../
```
Lastly, you can get the output of aspect and sentiment of sentences and validate the output.
```
!python scripts/predict_test.py --models-path {"2021-11-13-01:48:53_checkpoint.pt"} \
                        --max-length 160 \
                        --bert-type {"base-bert"} \
                        --real-test-path {"data/semEval2014.tsv"} \
                        --fake-test-path {'data/faker.tsv'}
```
