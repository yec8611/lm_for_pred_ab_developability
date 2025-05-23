# Using protein foundation models to predict antibody expression 
This is a pilot study for using protein language models to predict antibody developability.
We aimed to model various developabilities, with yield (expression in Hek293) being the first target property. We compared the performace of multiple ESM2 models with different paramter sizes, as well as different fine tuning strategies (frozen weights + regression head vs parameter efficient fine tuning (LoRA)). Further, we explored using antibody specific protein language model (aka. IgBERT) and its fine-tuned version to predict antibody yield and compare the results with the ESM2 series. We used a dataset for training from [Jain et al 2017 PNAS](https://www.pnas.org/doi/10.1073/pnas.1616408114), and another dataset from [Koenig et al 2017 PNAS](https://www.pnas.org/doi/10.1073/pnas.1613231114?url_ver=Z39.88-2003&rfr_id=ori%3Arid%3Acrossref.org&rfr_dat=cr_pub++0pubmed).

## Impact of dataset size on model performance
We used the two datasets mentioned above to evaluate model performace on prediction of protein yield. There is drastic improvement of prediction with more data points (Koenig paper).

<img width="397" alt="Screenshot 2025-05-14 at 3 55 53 PM" src="https://github.com/user-attachments/assets/421ee36f-cf29-44cc-a724-6827f6d371b0" /> 
<img width="414" alt="Screenshot 2025-05-14 at 3 56 50 PM" src="https://github.com/user-attachments/assets/684ba675-0337-4ccc-bce4-6be745c35907" />

The left figure is the result of training ESM models on Jain dataset (highly unstable and variable; the best spearman ρ is 0.32 using ESM2-650M) and the right is training on Koenig dataset (the best ρ is 0.81 with LoRA-fine tuned ESM2-650M).

Below are more detailed training results.


## Use ESM2 series to predict antibody yield
### train with Jain dataset (~200 labels)
![image](https://github.com/user-attachments/assets/75274291-de18-4655-a313-9c79ede82fce)
Base: use pretrained weights with addition of a regression head.

LoRA: LoRA fine tuning plus regression head.

LoRA improves performance with ESM2-8M and ESM-150M but not others. The highest performance was with base ESM2-650M and LoRA fine-tuning had no improvement or made it worse. 

Conclusion: 1. big training variance due to small data size; 2. 8M -> underfit -> not much to adapt for LoRA; 3. 150M + LoRA -> best LoRA effect; 4. spearman coefficient climbs from 8M to 650M but dips at 3B (bigger models encode better latent information but may inflate noise when it's too big, causing degradation); 5. should collect more data.

### train with Koenig dataset (~3k labels)
![image](https://github.com/user-attachments/assets/d70f7d68-414c-4d5d-bde7-5ff4faa43894)
Huge improvement with more data points and more stable trainig with lower variance. ESM-150M is the sweet spot without fine tuning for 3k labels; LoRA drastically improves prediction and reduces Loss for every model size, with 650M having the highest performance in ranking (LoRA unlocks large model gain with more training data).


## Use IgBERT to predict antibody yield
### train with Jain dataset
![image](https://github.com/user-attachments/assets/6b698bbb-4a8e-4a6e-b2a5-ef480ca218a4)
IgBERT was fine-tuned on antibody chains, so its latent space already encodes many developability motifs, thus predicting with ~15 % lower MSE and roughly same spearman coefficient despite a smaller backbone (420M vs 650M). LoRA fine-tuning on IgBERT hurts both MSE and spearman correlation -> we injected unnecessary capacity into an already well-aligned representation.

### train with koenig dataset
<img width="401" alt="Screenshot 2025-05-23 at 12 52 30 AM" src="https://github.com/user-attachments/assets/a7d78819-aeac-440d-821c-7dd223f721eb" />
<img width="400" alt="Screenshot 2025-05-23 at 12 52 48 AM" src="https://github.com/user-attachments/assets/ed3d1f92-c72f-4a36-9186-62e8a9a38dd8" />
LoRA significantly improves performance on ranking; error bars are much smaller than training with smaller datasets. For expression prediction problem, IgBERT with domain-specific priors does NOT bring advantages compared to general model ESM, even with LoRA.




## Use embeddings from protein language models for classical machine learning
### train with Jain dataset
![image](https://github.com/user-attachments/assets/930dc76b-896c-4d9f-86bf-155fa331606c)
From 8M to 150M we have clear gain in prediction which plateaued between 150M and 650M. Bigger model (3B) has representation noise that cannot be averaged out by classical ML. The best performing model-algorithm in terms of correlation is ESM2-650M + XGBoost (ρ = 0.42), while ESM-150M + Randomforrest is both performant and cost-effective (ρ = 0.41). For predicting yield, domain-specific prior in IgBERT does not help in this type of model structure compared to ESM.

### train with koenig dataset
![image](https://github.com/user-attachments/assets/e55ad3e4-b019-4299-8cfb-a7544d72be52)
Boost in performance in both loss and spearman correlation compared to training on small dataset. ESM-35M + random forrest is the best combination in terms of loss and ranking (ρ = 0.73). IgBERT with SVR is competitive with a correlation of 0.71. Note that SVR works better with IgBERT probably because of its layer norm mechanism (ESM embeddings may need standardization before fed into SVR to achieve better performance).
