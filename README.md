# Classical Chinese Quatrain Generation
#### Sean Y. Li
## Introduction: GANs
![GAN structure model](./images/GANmodel.png)
A Generative Adversarial Network (GAN) is a type of neural network whose two main components, the generator and discrminator, are at odds with each other. Generators works similarly to autoencoders and also like the decompression part of compression algorithms for things like Winrar (Did you pay for your license?) or zip files. Except instead of decompressing a compressed file, it generates something random because the algorithm was used on random noise. For the MNIST dataset often used in simple GAN demonstrations, the process would be taking random noise and outputting a matrix of 28x28. The discriminator is more simple, binary classification neural network, tasked with determining whether the data is real or fake. With each loop, the generator will change its weights and try to generate images to fool the discriminator. The discriminator, which increasingly has a harder time telling the difference between real and fake images and also adjusts its weights, pitting the two networks against each other. Thus the combination is called an “adversarial” network. 



![cat](./images/cat1.jpg) ![cat](./images/cat2.jpg) ![cat](./images/cat3.jpg) ![cat](./images/cat4.jpg)

![cat](./images/cat5.jpg) ![cat](./images/cat6.jpg) ![cat](./images/cat7.jpg)
https://thiscatdoesnotexist.com/

With enough training, a GAN can generate very realistic looking data/images. The cats on the right were all made through a GAN. While the background of the last picture seems a little distorted, the cats themselves are indistinguishable to pictures of real cats.



## Background: Quatrains
Classical Chinese *juéjù* 絕句 are [quatrains](https://en.wikipedia.org/wiki/Jueju) of either five or seven syllables popularized in the Tang dynasty. Below is an example of one of the more famous ones.

春曉 Spring Dawn
[孟浩然 Meng Haoran](https://en.wikipedia.org/wiki/Meng_Haoran)

春眠不覺曉 [spring] [sleep] [not] [aware] [dawn]
\
處處聞啼鳥 [place] [place] [hear] [chirp] [bird]
\
夜來風雨聲 [night] [come] [wind] [rain] [sound]
\
花落知多少 [flower] [fall] [know] [many] [few]

In Spring one sleeps, unaware of dawn;
\
everywhere one hears chirping birds.
\
In the night came the sound of wind and rain;
\
who knows how many flowers fell?

You might think at this point, why try to use something that normally generates images to generate text? Well, if you look at it from another angle, these quatrains actually resemble images a lot. Take this comparison with the famous MNIST dataset that GAN demonstrations often use.

Quatrain Features:
* Set dimension (7&times;4 and 5&times;4)
* Finite character possibility (characters in training data)

MNIST Features:
* Set dimension (28x28)
* Finite pixel intensity (255)

The additional parameters of a strict meter format and rhyme scheme only give a discriminator more parameters to scrutinize the generator creations. In additon, the lack of inflection and loose word order in Chinese means the model doesn’t need to worry much about grammar of the output.


## Problem Statements
Using this framework I came up with 3 problem statements.
1. Can a GAN which is normally used to generate images of a fixed size, be used to generate poetry of a fixed size?
2. Can such a GAN learn meter constraints? 
3. Can such a GAN learn a rhyme scheme?


## Data Formats
#### Character Metadata
| Key         | Example | Description                                                                                                       |
|-------------|--------:|-------------------------------------------------------------------------------------------------------------------|
| index       |    1003 | index of the character                                                                                            |
| char        |      地 | character                                                                                                         |
| tone        |      去 | [character tone](https://en.wikipedia.org/wiki/Four_tones_(Middle_Chinese)) (平: level tone 上: rising tone 去: departing tone 入: entering tone)                              |
| rime        |      至 | rime (character indicating same final) from [_Guangyun_](https://en.wikipedia.org/wiki/Guangyun) rime dictionary                                           |
| ipa         |     dij | Middle Chinese reconstruction in [International Phonetic Alphabet](https://en.wikipedia.org/wiki/International_Phonetic_Alphabet) notation                                                |
| tone_class  |       H | tone as letter (L: level tone, X: rising tone, H: departing tone, E: entering tone)                               |
| pinyin      |      dì | Pinyin romanization for modern Mandarin pronunciation                                                                                     |
| jyutping    |    dei6 | Jyutping romanization for modern Cantonese Pronunciation                                                                                    |
| hangul      |      지 | Modern Korean Pronunciation                                                                                       |
| rime_index  |       3 | index of major rime group                                                                                         |
| meter_class |       0 | Classical Chinese meter is determined by being either 1 for 平 level tone or 0 for 仄 oblique tone (all non-level tones)  |                                                                                 
#### Poem format
| additional for hepta- | pentasyllabic |   rhyme  |
|:---------------------:|:-------------:|:--------:|
|     ○○                |     ●●○○●     |          |
|     ●●                |     ○○●●○     |     ✓    |
|     ●●                |     ○○○●●     |          |
|     ○○                |     ●●●○○     |     ✓    |

Above is and example of a type of possible meter for Chinese Quatrains. The white circles indicate level tones (meter class 1) and the black circles indicate oblique tones (meter class 0) for every position in the poem. 

## Data Source
Poetry data was from a [Chinese Poetry GitHub repository](https://github.com/chinese-poetry/chinese-poetry) and character metadata was scraped from an [online rime dictionary database](https://ytenx.org/). Modern pronunciations generated courtesy of [pinyin](https://pypi.org/project/pinyin/) (Mandarin), [PyJyutping](https://pypi.org/project/pyjyutping/) (Cantonese), and [hanja](https://pypi.org/project/hanja/) (Korean) Python packages.

## Formatting, Cleaning, and Character Dictionary Building
The initial data is from a collection of .json files containing rough ~300,000 poems of different types from the Tang and Song dynasty. A function was used to determine if an entry was a quatrain, and to clean and add validated quatrains to a DataFrame. The above texts were vectorized using SKlearn CountVectorizer and a character list was generated from it. Non word symbols found here were used to refine the regex used to clean the poetry. Rerunning the quatrain extractor with the updated regex yielded 16785 pentasyllabic poems and 86578 heptasyllabic poems after dropping ~4000 duplicates. Next Beautiful Soup was used to extract the metadata from the online rime dictionary. Modern pronuciations were added and a rime index was added consolidating 208 individual rimes into 16 major rime groups. How this works is the rime is simply everything that comes after the initial consonant, so words with different full rimes might still rhyme because the last few parts of their rime are still the same. An analogy in English would be "spent" and "silent" rhyming, but not in the same way as "silent" and "violent" rhyme more completely. 

With the character list I removed around 16000 poems with characters that I didn't have metadata on. These were mostly obscure and/or variant characters that require interpretation. I rated the rest of the poems for rhyme and meter adherence. All the poems rhymed correctly and hade meter adherence of exactly half or complete adherence. I believe this is due to the rules surrounding intentional meter breaks inverting the meter for other parts of the poem. I didn't think it would be wise to try to edit my meter scoring function for these complex rule. I wanted my model to be able to make something, not try to be the next [Li Bai](https://en.wikipedia.org/wiki/Li_Bai) or Shakespeare. After checking the intergrity of my poems, I chose to focus on the 62211 heptasyllabic 7&times;4 poems which comprised the majority of the quatrains.

## Modeling
The basic training loop for the GAN looked like this:
* Generating Poems
  * Generate random noise in generator input shape
  * Run noise through generator to produce poems
* Transform Fake Poems
  * Add on rhyme and meter scores for fake poems
* Discriminator Training Phase
  * Concatenate fake and real poems and assign 1 for real poems and 0 for fake poems to y array
  * Train discriminator on data
* Generator Training Phase
  * Turn off discriminator training
  * Generate noise and run model
* Loop
  * Turn discriminator training back on and run through steps again

## Roadblocks
Tensorflow GPU wasn't working for me. Although my "image" size was considerably smaller, so was my computing power. For reference, the type of GAN used to generate the above cats take approximately the below training times for the default configuration using Tesla V100 GPUs.

| GPUs | 1024&times;1024  | 512&times;512    | 256&times;256    |
| :--- | :--------------  | :------------    | :------------    |
| 1    | 41 days 4 hours  | 24 days 21 hours | 14 days 22 hours |
| 2    | 21 days 22 hours | 13 days 7 hours  | 9 days 5 hours   |
| 4    | 11 days 8 hours  | 7 days 0 hours   | 4 days 21 hours  |
| 8    | 6 days 14 hours  | 4 days 10 hours  | 3 days 8 hours   |

## Alternative Model
Using LSTM based text prediction, I built a simpler neural network that takes in a sequence of 7 characters and spits out the predicted next character.  To generate a poem, you’d need a seed text of the first line randomly generated or of your choosing. Then you take the last six characters of your seed text and the predicted character and you plug that back in the model until you have 28 characters. Training this model on a CPU took over 8 hours, but because the neural network cannot take into account it's position in the poem, it has no way of learning rhyme and meter. It might be possible to sequentially generate characters using a separate neural network for that specific position in the poem, thus possbibly allowing each network to learn the correct meter class of that position needs. But training and tuning neural networks would still take considerable time.

## Findings
Yes, poems and image data superficially similar but with “intensity” comes “complexity”.  Trying to learn tone classes and rimes from nothing takes a lot of training. It is a much simpler to try to find simpler patterns for values between 0-255 for something like edge detection than trying to learn the tone classes and rimes of 7980 characters from scatch. Although LSTMs work much better than simple Dense neurons for language data, they still don't work nearly as well as transformers.

## Next Steps
I do want to try running my GAN at some point, but it would realistically need to be done in the cloud. If I could incorporate 


#### Sources
* [Chinese Poetry GitHub Repository](https://github.com/chinese-poetry/chinese-poetry)
* [Online Rime Dictionary Database](https://ytenx.org/)
* [pinyin](https://pypi.org/project/pinyin/)
* [PyJyutping](https://pypi.org/project/pyjyutping/)
* [hanja](https://pypi.org/project/hanja/)


