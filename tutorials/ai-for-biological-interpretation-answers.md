# Tutorial: cerebral melanoma brain biopsies (PXD007592) — Answer sheet

Answers to the numbered questions in [ai-for-biological-interpretation.md](ai-for-biological-interpretation.md).

**Q1**: MLMarker feature coverage varies quite a bit across these six samples, roughly 6% to 30%. Does any of them get flagged with a red dot in the "Low" column, and would you turn on the penalty factor for them?

**A1**: No, none of them are flagged. The warning only fires below 5% coverage, and even the sparsest sample here (good_responder_1_1, ~6.3%) sits just above that line. These are still solid brain tissue biopsies, not biofluids or cell lines, so the missing proteins are more likely genuine biology than assay sparsity, and we leave the penalty factor off. (We'll come back to this decision in section 4, once one particular sample's prediction looks a bit too dependent on absence.)

**Q2**: The result heatmap already shows a clear trend. What stands out?

**A2**: Poor responders show a much stronger brain similarity than good responders. Good responders instead show a signature dominated by monocytes, a very different, immune-cell-like profile for tissue that is supposed to be brain.

**Q3**: Putting Figs A to C together, what does this tell you about the non-brain-predicted samples specifically?

**A3**: Their predictions rest on a handful of very highly abundant proteins in an otherwise sparser feature space, rather than on a broad, evenly covered set of proteins the way the brain predictions do. That's not necessarily wrong, but it's a good reason to look more closely before trusting those particular calls: a prediction built on a few loud proteins is more fragile than one built on hundreds of moderate ones.

**Q4**: What is its second-highest predicted tissue? Does that make biological sense?

**A4**: Pituitary gland, at 14.8%. That's a sensible runner-up: it's also a neuronal, brain-adjacent tissue, so some overlap in protein signature is expected rather than alarming.

**Q5**: Comparing the two force plots, what do you observe?

**A5**: When looking at the SHAP values for the top 10 pro and top 10 con proteins, the prediction of brain has a strong cumulative positive value, and only a small cumulative negative value. Conversely, the prediction of pituitary gland has a much larger cumulative negative value, even though the positives still outweigh the negatives. In other words, the evidence against pituitary gland (in favor of brain) is stronger and more one-sided than the evidence against brain (in favor of pituitary gland), which is exactly why brain wins by a wide margin while pituitary still shows up as a plausible runner-up.

**Q6**: Compare its monocyte force plot with its brain force plot. What happens? Pay special attention to the top pro protein for both tissue predictions.

**A6**: The monocyte prediction is driven by a small number of highly monocyte-specific proteins with large SHAP values, one of them being P98160, which is specific for both tissues but far more so for monocytes, with dramatically different axis scales between the two plots. Compare that to poor_responder_12_1's Brain force plot, which is supported by many moderate contributors rather than a handful of extreme ones: a single strong, specific protein can tip a whole sample's call in a direction the underlying tissue doesn't actually support.

**Q7**: Recall from Q1 that this sample sat at 6.3% coverage, just above the 5% cutoff for the automatic warning. Given what you're seeing here, how could you balance this out?

**A7**: Turn on the penalty factor for this sample after all. A prediction built almost entirely on a huge pile of absent proteins, in a sample that already has very low coverage, is a textbook case for down-weighing missing features, even though the app's automatic 5% threshold didn't flag it. The threshold is a useful default, not a substitute for looking at the actual result.

**Q8**: What is this protein, and how does it end up supporting a monocyte call?

**A8**: P98160 is Perlecan (HSPG2), a basement-membrane heparan sulfate proteoglycan. According to the Human Protein Atlas it has low tissue and cell-type specificity on its own, but its role in vascular and endothelial basement membranes means it turns up in samples like monocytes (which travel closely with the vasculature), and it can also be picked up in CSF, which is presumably where its brain association comes from too. MLMarker's training data is built from solid tissues and specific isolated cell pools, and it doesn't include a comprehensive whole-blood or vascular/endothelial reference profile. So within the bounds of what the model has actually seen, Perlecan showing up looks like a monocyte signal, even though biologically it's a much less specific protein than that. This is a useful reminder that MLMarker's "specificity" is always relative to its training set, not to biology in general.

**Q9**: Of the 714 proteins contributing positively to the brain prediction, 151 are present and 563 are absent. Is it surprising that so many absent proteins contribute positively?

**A9**: No, and this is worth sitting with for a moment, because it's easy to misread at first. An absent protein isn't "brain-specific" in any direct sense; it's informative by elimination. A missing liver-metabolism enzyme, for instance, argues against liver, which by process of elimination makes brain (among others) relatively more likely. The model doesn't need a protein to be present to use it as evidence, it just needs the pattern of what's missing to be different between tissues.

**Q10**: What does the ORA on the 151 present proteins give you?

**A10**: Mostly genuinely brain-related biological processes, terms you'd expect from an actual brain proteome, such as synaptic signaling or neuron projection. This is a coherent list because these are real, measured brain proteins doing real brain jobs.

**Q11**: And on the 563 absent proteins? More terms or fewer than you'd expect, and does the pattern match your expectations from Q9?

**A11**: Only a few more terms come back, despite roughly five times as many input proteins, and the terms that do show up are scattered across unrelated categories rather than clustering around one theme. This is the flip side of Q9: this protein list isn't "563 things brain doesn't do", it's a mixture of muscle proteins, liver proteins, immune proteins and more, each one absent for its own tissue-specific reason. Grouped together they don't share a common biological process, so ORA can't find one, even though every individual protein was genuinely useful evidence to the model. Presence-based evidence tends to be specific and enrich cleanly; absence-based evidence tends to be diffuse, informative in aggregate but not as a coherent biological story.

**Q12**: Now switch the PCA mode to **Tissue-Specific SHAP** and select Monocytes. This clusters the samples on SHAP values for the selected tissue prediction instead of simple protein abundances. What changes?

**A12**: A good responder that previously clustered apart now sits close to the poor responders, because on this specific axis, none of these samples are actually leaning on the monocyte signal, so they look similar again. This is the key distinction between the two PCA modes: clustering on protein abundance reflects overall sample composition, while clustering on a tissue-specific SHAP profile reflects only what's driving that one prediction. Two samples can look different overall, while also looking similar from one tissue's point of view, or the reverse.

**Q13**: The test reports U = 9 and p = 0.1, not significant at the usual p < 0.05 threshold, even though the two groups are visually about as separated as they could possibly be. Why?

**A13**: U = 9 out of a maximum of 9 (3 poor responders × 3 good responders = 9 possible pairs) means every single poor-responder value ranks above every single good-responder value, the strongest separation this test can report. The catch is sample size, not effect size. With only three samples per group, there are exactly 20 ways to split six ranked values into two groups of three, and complete separation happens in only one of those arrangements per direction, giving an exact two-sided p-value of 2/20 = 0.1. That is the smallest p-value this test can ever produce with n = 3 vs n = 3, no matter how clean the separation is; you cannot reach p < 0.05 here even with a perfect result. This is a power problem, not evidence against a real difference: the same complete separation with four samples per group would already reach p ≈ 0.029. The lesson generalizes well beyond this dataset: a non-significant p-value with a tiny sample size tells you the test was underpowered, not that the groups are the same.
