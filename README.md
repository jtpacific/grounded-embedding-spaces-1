# Grounded Embedding Spaces

This repository contains code for creating visually grounded word embedding spaces used in the paper "Cortical Representations of Concrete and Abstract Concepts are Grounded in Visual Features" by Jerry Tang, Amanda I. Lebel, and Alexander G. Huth.  

## Usage

1. Download [data](https://utexas.box.com/shared/static/9s0kymmfpx5vc7lg8o2l47uxjim8bw2t.zip) and extract content into new `data/` directory. 

2. Extract visual embeddings from images (if using custom images). The images corresponding to each visual word should be stored in a subdirectory of `imgs/`. If the subdirectory names are different from the words (e.g. if subdirectory names are WordNet IDs), a mapping between each word and its image subdirectory name should be provided in `data/word_dir_mapping.txt`. 

```bash
python3 create_visual_embeddings.py data/visual_embs.npz -layer fc1 -mapping data/word_dir_mapping.txt
```

Alternatively, the visual words and corresponding WordNet IDs used in Tang et al. are provided in `data/word_dir_mapping.txt`, and the visual embeddings (extracted from layer fc1) used in Tang et al. are provided in `data/visual_embs.npz`. Unfortunately, due to copyright restrictions we are not able to share the original images used to create these embeddings.

3. Create the visually grounded embedding space. Given the visual embeddings of visual words, a sensory propagation method creates visually grounded embeddings of nonvisual  words. The amodal and grounded embedding spaces are represented by their covariance matrices and saved to the specified output file. 

```bash
python3 create_semantic_priors.py data/visual_embs.npz semantic_priors.npz
```

4. Create concreteness scores for weighting amodal and grounded embeddings. The [Brysbaert Concreteness Ratings](https://www.ncbi.nlm.nih.gov/pubmed/24142837) are provided in `data/conc_ratings.npz`. This function creates concreteness scores for each word in the embedding space by processing concreteness ratings and interpolating scores for missing words. 

```bash
python3 create_concreteness_scores.py semantic_priors.npz conc_scores.npy
```

5. Use grounded embedding spectrum. The GroundedSpectrum class loads the saved semantic priors and concreteness scores. 

```python
In [1]: from GroundedSpectrum import GroundedSpectrum
In [2]: gs = GroundedSpectrum('semantic_priors.npz', 'conc_scores.npy')
In [3]: gs.similarity('doctor', 'athlete', b = -10) # amodal similarity
Out[3]:
0.02106039500201583
In [4]: gs.similarity('doctor', 'athlete', b = 10) # visually grounded similarity
Out[4]:
0.4563762578004484
In [5]: gs.grounding_words('idyllic')[:10] # most strongly associated visual words under the amodal embedding space
Out[5]:
[('gorges', 0.3111969206758748), 
 ('beauty', 0.3065462045460804), 
 ('garden', 0.2967788022308049), 
 ('cottage', 0.2866977052313361), 
 ('family', 0.26388145847297373)]
In [6]: gs.sigma(0)
Out[6]:
array([[ 1.        ,  0.69081572,  0.3095305 , ..., -0.24495695,
         0.24722239, -0.22175797],
       [ 0.69081572,  1.        ,  0.53911375, ..., -0.26954793,
         0.47671069, -0.24276763],
       [ 0.3095305 ,  0.53911375,  1.        , ..., -0.27715573,
         0.62269407,  0.01050194],
       ...,
       [-0.24495695, -0.26954793, -0.27715573, ...,  1.        ,
        -0.32578409, -0.32810192],
       [ 0.24722239,  0.47671069,  0.62269407, ..., -0.32578409,
         1.        , -0.04632105],
       [-0.22175797, -0.24276763,  0.01050194, ..., -0.32810192,
        -0.04632105,  1.        ]])
```
