# HTK demo

overview of HTK usage from feature extraction to word recognition.

install HTK3.4.1 from [http://htk.eng.cam.ac.uk/](http://htk.eng.cam.ac.uk/)


## HCopy: extract MFCC features from WAVs

require: create a feature definition as `script.hcopy`, place WAVs in `speech` and labels in `labels`.

``` sh
mkdir mfcc
ls speech | sed -e "s/\.[^.]*$//g" | xargs -I {} echo speech/{}.wav mfcc/{}.mfc > script.hcopy
HCopy -C config.hcopy -S script.hcopy
```

result: `mfcc/*.mfc`

## HInit: initialize HMMs with training file list

require: initial hmm definitions as `*.hmm` in `proto`

(todo: automate the creation of initial hmm definitions)

``` sh
ls proto | sed -e "s/\.[^.]*$//g" > hmmlist.txt
ls mfcc | xargs printf "mfcc/%s\n" > trainlist.txt
mkdir hmm0
cat hmmlist.txt | xargs -I {} HInit -T 1 -S trainlist.txt -M hmm0 -H proto/{}.hmm -l {} -L label {}
```

result: `hmm0/*.hmm`

## HRest: train HMMs

``` sh
mkdir hmm1
cat hmmlist.txt | xargs -I {} HRest -T 1 -S trainlist.txt -M hmm1 -H hmm0/{}.hmm -l {} -L label {}
```

result: `hmm1/*.hmm`

## HParse: parsing grammar into lattice network

require: a grammar definition as `grammar.txt`

``` sh
HParse grammer.txt net.slf
```

result: `net.slf`

## HVite: recognize words from MFCC features

require: a dictionary of words as `voca.txt`

from one MFCC feature `mfcc/hai1.mfc`

``` sh
HVite -T 1 -H hmmsdef.hmm -i reco.mlf -w net.slf voca.txt hmmlist.txt mfcc/hai1.mfc
cat reco.mlf
```

from MFCC feature sets `trainlist.txt`

``` sh
HVite -T 1 -S trainlist.txt -H hmmsdef.hmm -i rec.mlf -w net.slf voca.txt hmmlist.txt
cat rec.mlf
```

## HResults:

warning: just reuse trainlist.txt as evallist here

``` sh
echo '#!MLF!#' > ref.mlf
echo '"label"' >> ref.mlf
ls label | xargs printf "%s\n" >> ref.mlf
HResults -T 1 -e "???" sil -I ref.mlf -L label hmmlist.txt rec.mlf
```

result:

```
====================== HTK Results Analysis =======================
  Date: Thu Aug 25 01:53:15 2016
  Ref : ref.mlf
  Rec : rec.mlf
------------------------ Overall Results --------------------------
SENT: %Correct=60.00 [H=6, S=4, N=10]
WORD: %Corr=60.00, Acc=60.00 [H=6, D=0, S=4, I=0, N=10]
===================================================================
```
