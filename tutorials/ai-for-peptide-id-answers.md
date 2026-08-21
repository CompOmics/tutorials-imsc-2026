# Tutorial 1: Data-driven rescoring of peptide identifications — Answer sheet

Answers to the numbered questions embedded in the tutorial notebook. "Try it" assignments are not included here, as they have no single correct answer.

## Section 1.1 — Basic statistics

**[1.1a]** *The number of PSMs is larger than the number of unique peptides. Name two reasons a single peptide can be matched more than once.*

First, the same peptide can be observed at more than one precursor charge state, which produces a separate PSM (and a separate precursor) for each charge. Second, in DDA the same precursor can be selected for fragmentation repeatedly over the course of the LC gradient (co-eluting isotopes, repeat sampling of an abundant peptide), producing multiple independent spectra, and therefore multiple PSMs, for the same peptide.

**[1.1b]** *As the counts above show, a sizeable share of peptides map to more than one protein: each is equally good evidence for any of those proteins, so the search alone cannot say which protein is truly present. What is this phenomenon called, and why does it complicate reporting "identified proteins"?*

This is peptide degeneracy, and the resulting ambiguity is the protein inference problem. A shared peptide is equally strong evidence for every protein it could have come from, so the search engine cannot decide, from that peptide alone, which specific protein (or proteoform, or homolog) is actually present in the sample. Reporting "identified proteins" therefore requires an additional inference step, typically grouping proteins with identical or overlapping peptide evidence into protein groups, or applying parsimony rules to report the smallest set of proteins that explains all observed peptides.

## Section 1.2 — Score distributions and the FDR threshold

**[1.2a]** *Which PSMs are accepted at the 1% FDR threshold: the ones to the left of the dashed line, or the ones to the right?*

The ones to the right. A higher Sage score indicates a better match, so the accepted PSMs are those scoring at or above the threshold, to the right of the dashed line.

## Section 2.1 — The chromatogram, for context

**[2.1a]** *The density of identified PSMs roughly follows the chromatogram's shape, but not perfectly. Where would you expect relatively more or fewer identifications: near the start and end of the gradient, or in the middle? Why?*

Relatively fewer identifications near the start and end, more in the middle. A reversed-phase gradient separates peptides mainly by hydrophobicity, and most tryptic peptides fall in a moderate hydrophobicity range that elutes around the middle of the gradient. Very hydrophilic peptides elute early, near the void volume, before the gradient has had much separating effect, and very hydrophobic peptides elute late, where fewer peptides of the sample actually fall. Peptide density, and therefore identification density, consequently peaks in the middle.

## Section 2.2 — Predicted vs. observed retention time

**[2.2a]** *Click "False" and "True" in the legend to toggle targets and decoys on and off separately. What is the key difference between how the two are distributed in this scatter plot?*

Targets show two populations: one lying close to the diagonal, where predicted and observed RT agree, and one scattered with no particular relationship between predicted and observed RT. Decoys show only the scattered population. This is expected: a correct target PSM has a real peptide with a real, predictable retention behavior, so agreement with the prediction is possible. A decoy is not a real peptide, so there is no physical reason for its predicted and observed RT to agree, and none do beyond chance.

## Section 2.3 — RT error: target vs. decoy

**[2.3b]** *Some decoy PSMs also show a small RT error. Is this a problem for the model, or expected? Why?*

Expected, not a problem. With tens of thousands of decoy sequences, some will, purely by chance, have a hydrophobicity pattern similar enough to produce a small RT error, even though the decoy itself is not a real peptide. This is exactly why RT error is not used as the sole criterion, but combined with other independent features: a chance agreement on one feature is far less likely to also hold across several unrelated features at once.

## Section 3.4 — Fragmentation similarity: target vs. decoy

**[3.4a]** *We now have three separating distributions: search engine score, RT error, and fragmentation similarity. Do you expect them to agree on which PSMs are correct, or to sometimes disagree? What would a disagreement look like in practice?*

Mostly agree, but not always. The three features probe largely independent physical properties (fragment-matching quality, chromatographic behavior, and relative fragment intensity pattern), so a wrong match can satisfy one by chance without satisfying the others. A disagreement would look like a PSM with a high search engine score but a poor RT match or low fragmentation similarity, or conversely a modest search engine score paired with strong agreement on both RT and fragmentation. These disagreements are precisely the cases that benefit most from combining the evidence.

## Section 4.1 — Do the features agree with each other?

**[4.1a]** *Do you see PSMs that score well on one feature (RT error) but poorly on the other (fragmentation similarity)? What might that combination mean for such a PSM's true identity?*

Yes, such PSMs are visible in the scatter plot. A PSM scoring well on one feature but poorly on the other is a borderline or ambiguous case: it received strong support from one independent line of evidence but not the other, so its identity is less certain than a PSM supported by both. These are exactly the PSMs where combining features into a single rescored value should have the largest effect, positive or negative, since neither feature alone would have given a confident answer.

## Section 4.2 — Old score vs. new score

**[4.2a]** *Four quadrants are formed by the two threshold lines. Which quadrant contains PSMs that were rejected before but are accepted after rescoring? Which contains the opposite case?*

The upper-left quadrant (below the old-score threshold, above the new-score threshold) contains PSMs that were rejected before rescoring but are accepted after: these are the gained identifications. The lower-right quadrant (above the old-score threshold, below the new-score threshold) contains the opposite case: PSMs accepted before rescoring but rejected after, the lost identifications.

## Section 4.3 — Gained, lost, and retained identifications

**[4.3a]** *Some PSMs that passed the 1% FDR threshold before rescoring are `lost` afterwards. These are not simply removed at random: look at their `neg_log_rt_error` and `rescoring:spec_pearson_norm` values. What do they have in common? Is losing them a good or a bad outcome for the overall dataset?*

Lost PSMs typically show weak values for RT error, fragmentation similarity, or both, despite having passed on search engine score alone. Their original acceptance was likely a coincidentally good spectral match rather than a well-supported identification. Losing them is a good outcome for the dataset overall: at the same nominal 1% FDR, the accepted set becomes more precise, since matches whose only support was the search engine score, unconfirmed by independent physicochemical evidence, are removed, even though a handful of individual identifications are lost in the process.
