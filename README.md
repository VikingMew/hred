##### An implementation of the paper  [Building End-To-End Dialogue Systems Using Generative Hierarchical Neural Network Models](https://arxiv.org/abs/1507.04808)

#### Results

The model is *able to replicate the results* of the paper. 

| Model           | Test Perplexity | Training Loss | #of epochs |
|-----------------|-----------------|---------------|------------|
| HRED* + Bi + LM | 33.248          | 3.322         | 25         |
| HRED            | 35.128          | 3.334         | 8          |

Model 1
`python3.6 main.py -n full_all -e 25 -tc -lr 0.0001 -drp 0.4 -lm -bi -bms 20 -bs 100 -seshid 300 -uthid 300 -nl 2`

Model 2
`python3.6 main.py -n full_final2 -tc -bms 20 -bs 100 -e 80 -seshid 300 -uthid 300 -drp 0.4 -lr 0.0005`

 - We notice over fitting on the validation loss (patience 3) from epoch 8 onwards for the second model and from epoch 24 (smaller learning rate) for the 1st one 
 - Training time is about 9 mins per epoch on a Tesla Geforce GTX Titan X consuming about 4GB of GPU RAM
 - The model checkpoint binary can be obtained from [here](https://nofile.io/f/oXmX0zozNHs/full_final2_mdl.pth)
 - Some sample beam search result
 - Test set results (partial with ground truth) available [here](https://github.com/hsgodhia/hred/blob/master/c.out)
 ```
("  i don ' t know .  ", -12.539569854736328), 
("  i ' m sorry .  ", -12.661258697509766), 
("  i ' m sorry , <person> .  ", -16.370222091674805), 
("  i ' ll be right back .  ", -17.198185920715332),
("  i don ' t believe you .  ", -17.357797622680664), 
("  i ' m not sure .  ", -16.219301223754883),
("  <person> , i ' m sorry .  ", -17.79110860824585),
("  i ' m sorry , sir .  ", -18.12678337097168), 
("  i didn ' t know .  ", -16.70194959640503), 
('  what are you talking about ?  ', -16.77923345565796), 
("  i don ' t think so .  ", -18.42203998565674), 
("  i don ' t believe it .  ", -18.48332691192627), 
("  i don ' t want to .  ", -18.522937774658203), 
("  what ' s your name ?  ", -17.191880226135254), 
("  i ' ll take care of myself .  ", -20.434056282043457), 
("  i don ' t know what to do .  ", -21.852730751037598), 
("  i didn ' t mean to .  ", -19.00902223587036), 
("  i don ' t know . <person> .  ", -20.844543933868408),
("  i ' d like to .  ", -18.216479301452637),
("  i don ' t know what to say .  ", -22.77810764312744)]
Ground truth [('  he hid them but i found them . <continued_utterance> <person> ?  ', 0)
```

#### Notes
 - Greedy decoding is used during training time if teacher forcing is disabled (by default we train with tc)
 - During inference time the MMI-antiLM score is computed as per equation (15) in Jiwei Li (Diversity Promoting Paper)
 - an LM loss is by default included (through an additional plain RNN) and this jointly trained with the other parameters, although not much difference in results are obtained and to speedup I often disable it
 - When processing the data, diverse sequence lengths in a given batch leads to better optimization so no sorting of training data
 - Validation/test perplexity is calculated using teacher forcing as we want to capture the true work log likelihood
 - Inference or generation is using beam search
 
#### Train

`python3.6 main.py -tc -e 100 -n full_tc -bms 20 -bs 80`

A brief list of options is given below, for a longer list please see main.py file
- tc says that use teacher forcing for the entire training procedure
- bms is the beam size for decoing used only during inference time, during training if teacher forcing is disabled greedy decoding is used
- n is a required parameter that gives a name to the model files
- bs is the batch size
- e is the number of epochs 
- test boolean switch says to run only inference mode
- btstrp flag gives a name of a pre-trained model which is used for parameter initializations instead of the default of gaussian mean 0 and standard deviationn 0.01
 
#### Sanity check:-

 - If you load a small training set, like 1000 training and 100 valid as here `train_dataset, valid_dataset = MovieTriples('train', 1000), MovieTriples('valid', 100)` and train to overfit the model converges to 0.5 training loss in 50 epochs with training command `python3.6 main.py -n sample -tc -bms 20 -bs 100 -e 50 -seshid 300 -uthid 300 -drp 0.4`
 
 - Some samples generated at inference as compared to ground truth (on the test set) are
   ```
   [("i don ' t know . ", -11.935150146484375), ("  i ' m in from new york . i came to see <person> .  ", -20.482309341430664), ("  i ' ll take you there , sir .  ", -16.400659561157227), ("  i ' m sorry , but no one by that name lives here .  ", -22.178613662719727), ("  i know it ' s none of my business --  ", -18.43322467803955), ("  i don ' t think you win elections by telling <number> percent of the people that they are .  ", -27.444936752319336), ("  you ' re going to break up with <person> , aren ' t you ?  ", -23.688961029052734), ("  i ' m afraid not .  ", -14.662097930908203), ("  i don ' t know , do you ? <continued_utterance> it ' s a <person> .  ", -25.888113021850586), ("  i ' ll be right back .  ", -15.958183288574219)]Ground truth [("  what ' s bugging her ?  ", 0)] 
   ```