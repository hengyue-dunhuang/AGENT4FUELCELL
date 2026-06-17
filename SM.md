\documentclass[12pt]{article}

\usepackage[margin=1in]{geometry}
\usepackage{booktabs}
\usepackage{array}
\usepackage{amsmath}
\usepackage{longtable}
\usepackage{hyperref}

\title{Supplementary Material for ``Agent-driven discovery of advanced high-entropy alloy electrocatalysts for next-generation fuel cells''}
\author{}
\date{}

\begin{document}

\maketitle

\section*{S1. Manual audit of Agent-extracted records}

To evaluate whether the Agent-generated dataset was sufficiently reliable for literature-level trend analysis, 30 records were randomly selected from the 104 curated HEA/MPEA fuel-cell electrocatalyst records. The audit subset was checked manually against the corresponding original articles. The checked fields included catalyst composition, parsed element list, reaction type, electrolyte, support, morphology, synthesis route, half-cell or device test mode, activity metric, stability metric and benchmark catalyst.

For each field, a true positive was assigned when the Agent-extracted value matched the source article after normalisation of aliases and units. A false positive was assigned when the extracted value was unsupported or incorrectly normalised. A false negative was assigned when a field was present in the source article but missing from the extracted record. Extraction accuracy was calculated as the fraction of audited fields that matched the manually checked source values.

\begin{table}[h]
\centering
\caption{Record-level audit metrics for 30 manually checked records.}
\begin{tabular}{lc}
\toprule
Metric & Value \\
\midrule
Precision & 0.9703 \\
Recall & 0.9423 \\
F1 score & 0.9561 \\
Field-level extraction accuracy & 0.9159 \\
\bottomrule
\end{tabular}
\end{table}

\begin{table}[h]
\centering
\caption{Field-wise extraction accuracy in the 30-record audit subset.}
\begin{tabular}{lc}
\toprule
Field group & Accuracy \\
\midrule
Composition and element list & 0.9327 \\
Reaction type and test mode & 0.9135 \\
Electrolyte and operating condition & 0.8654 \\
Support, morphology and synthesis route & 0.8269 \\
Activity metric and benchmark catalyst & 0.8365 \\
Stability metric and durability condition & 0.9231 \\
\bottomrule
\end{tabular}
\end{table}

The most common residual errors were incomplete durability-test descriptors, ambiguity between geometric-area-normalised and noble-metal-mass-normalised activities, and missing details for multi-step synthesis routes. These errors were corrected before the final trend analysis when they affected reaction assignment, composition parsing or quantitative performance correlations.

\section*{S2. Added interpretation beyond established catalyst motifs}

Several individual trends recovered by the Agent are consistent with established electrocatalysis knowledge, including Pt--Ni-containing ORR catalysts, Pt--Sn-containing alcohol oxidation catalysts and Ru-containing HOR catalysts. The added value of the Agent-derived dataset is the systematic cross-reaction quantification of these motifs in the HEA/MPEA literature rather than the discovery of an isolated binary composition.

The curated records show that Pt, Ni, Co, Fe, Cu and Pd behave as high-frequency hub elements. In contrast, Sn, Bi, Mo, Ru, Rh and Ir appear more selectively as reaction- or function-specific modifiers. This hub--modifier separation was used to interpret why the same composition family can be redirected toward ORR, MOR, EOR or HOR by sparse substitution rather than wholesale redesign.

The dataset also shows that five-element catalysts are the most common class, even though the formal HEA design space permits many higher-order combinations. This trend suggests that current HEA fuel-cell studies are constrained by synthesis, phase control, interpretability and benchmarking requirements. Finally, the cross-metric analysis indicates that intrinsic ORR activity metrics are more closely associated with reported fuel-cell power density than ECSA, particle size or half-wave potential alone, highlighting the need to report intrinsic and device-level metrics together.

\section*{S3. Agent prompt design and extraction protocol}

This section documents the prompt-controlled extraction protocol used to construct the literature-mined dataset. The purpose of including the prompt design is to make clear that the dataset was not produced by an unconstrained conversational query, but by a staged Agent workflow with explicit inclusion criteria, fixed output fields, conservative filtering and traceable intermediate records. The implementation used three main language-model calls after literature retrieval: a title--abstract prescreen, a full-text relevance check and a structured feature-extraction step. Full-text parsing was performed before the language-model extraction step, and both the parsed text and the extracted records were retained for later audit.

\subsection*{S3.1. PDF parsing and traceability}

For each candidate full text, the script first converted the PDF into markdown-like text using a document-layout parsing service. The parser request supplied the PDF as a base64-encoded file and disabled optional orientation correction, page unwarping and chart recognition so that the primary output used for extraction was the document text. The parser was called with up to three retries. When parsing succeeded, the full parsed markdown text was written to a local log file under a parser-log directory using the paper identifier and a sanitised title. These parser logs were used to diagnose extraction failures, verify whether missing values were absent from the source text or lost during parsing, and support manual audit of the Agent-generated records.

No authentication token or private service credential is reported here. Only the reproducible logic of the parsing and extraction workflow is described.

\subsection*{S3.2. Staged prompt architecture}

The Agent was designed as a narrow-domain extraction system rather than a general summarisation tool. Each prompt assigned the model the role of a fuel-cell electrocatalysis expert, specified the HEA/MPEA scope, and constrained the permitted response format. Table~\ref{tab:agent_prompts} summarises the three prompt stages.

\begin{longtable}{p{0.18\textwidth}p{0.34\textwidth}p{0.36\textwidth}}
\caption{Prompt stages used in the Agent extraction script.}
\label{tab:agent_prompts}\\
\toprule
Stage & Input & Output constraint and purpose \\
\midrule
\endfirsthead
\toprule
Stage & Input & Output constraint and purpose \\
\midrule
\endhead
Title--abstract prescreen & Paper title and abstract before PDF download or full-text processing. & Reply only \texttt{YES} or \texttt{NO}. The purpose was to avoid downloading and parsing papers that were clearly outside the HEA/MPEA fuel-cell electrocatalysis scope. \\
Full-text relevance check & The first 8000 characters of parsed paper text. & Reply only \texttt{YES} or \texttt{NO}. The purpose was to apply stricter inclusion criteria after the paper text became available. \\
Structured feature extraction & Parsed paper text and bibliographic metadata. & Return only a strictly valid JSON object with exactly the predefined keys. Missing or non-applicable values had to be reported as \texttt{null}; hallucinated values were explicitly forbidden. \\
\bottomrule
\end{longtable}

\subsection*{S3.3. Title--abstract prescreen prompt}

The prescreen prompt was intentionally permissive but still domain-specific. It was applied to the title and abstract before full-text parsing. The model was instructed to answer \texttt{YES} only when the title/abstract suggested that all three conditions were likely satisfied: the material was an HEA, high-entropy nanomaterial, MPEA or closely related multimetallic alloy catalyst; the application was fuel-cell-relevant electrocatalysis; and the paper likely reported experimental performance data. The prompt also explicitly excluded reviews, purely computational studies, water splitting, batteries, supercapacitors, sensors, photocatalysis and unrelated CO2 reduction papers.

The core prompt logic was:

\begin{verbatim}
You are a domain expert in fuel-cell electrocatalysis.
Determine whether this paper is likely useful for a small
literature-mined dataset about high-entropy alloy (HEA) or
multi-principal-element alloy catalysts for fuel-cell-related reactions.

Answer YES if the title/abstract suggests ALL of these are likely true:
1. The material is a high-entropy alloy, high-entropy nanomaterial,
   multi-principal-element alloy, or closely related multimetallic
   alloy catalyst.
2. The paper is about fuel-cell-relevant electrocatalysis, especially
   ORR, HOR, MOR, EOR, FAOR, AOR, fuel oxidation, or membrane-electrode/
   fuel-cell performance.
3. It likely reports experimental performance data such as mass activity,
   specific activity, ECSA, half-wave potential, power density, or durability.

Answer NO for reviews, purely computational papers, water splitting,
batteries, supercapacitors, sensors, photocatalysis, or unrelated CO2
reduction papers.

Reply with ONLY "YES" or "NO".
\end{verbatim}

If the model call failed during prescreening, the script conservatively allowed the paper to proceed to the next stage. This exception handling reduced the risk that a temporary API failure would remove a relevant paper before full-text screening.

\subsection*{S3.4. Full-text relevance prompt}

The full-text relevance check used a stricter prompt on the parsed paper text. To control cost and focus on the article front matter, abstract, introduction and experimental summary, the script used the first 8000 characters of the parsed text. A paper was included only when all four criteria were met: original experimental data; HEA, high-entropy nanomaterial, MPEA or closely related multimetallic alloy with at least four metallic elements; fuel-cell-relevant electrocatalysis or fuel-cell/MEA testing; and at least one quantitative electrochemical or device metric.

The core full-text inclusion prompt was:

\begin{verbatim}
You are a domain expert in fuel-cell electrocatalysis. Carefully evaluate
whether this paper should be included in a compact dataset for a paper titled
"Agent-Driven Literature Mining Decodes Design Heuristics for High-Entropy
Alloy Fuel-Cell Electrocatalysts".

Answer YES only if ALL conditions are met:
1. The paper reports original experimental data.
2. The catalyst is described as high-entropy alloy, high-entropy nanomaterial,
   multi-principal-element alloy, or a closely related multimetallic alloy
   with at least four metallic elements.
3. The application is fuel-cell-relevant electrocatalysis, especially ORR,
   HOR, MOR, EOR, FAOR, AOR, fuel oxidation, or fuel-cell/MEA testing.
4. The paper contains at least one common quantitative metric, such as ECSA,
   half-wave potential, mass activity, specific activity, current density,
   power density, or durability/retention.

Answer NO for reviews, purely computational/DFT papers, water splitting,
batteries, supercapacitors, sensors, photocatalysis, thermocatalysis, or
papers without quantitative electrocatalytic performance.

Reply with ONLY "YES" or "NO".
\end{verbatim}

In contrast to the prescreening step, an exception during the full-text relevance check was treated as non-relevant in the script. This made the final inclusion step stricter after full-text parsing had already been attempted.

\subsection*{S3.5. Structured extraction prompt and fixed schema}

For papers passing relevance screening, the extraction prompt asked the model to act as both a fuel-cell electrocatalysis expert and a data engineer. The output was constrained to a single JSON object with exactly the predefined keys. The prompt explicitly instructed the model to keep the dataset compact, prefer features commonly reported in ORR/HOR/MOR/EOR/FAOR/AOR and fuel-cell electrocatalyst papers, output \texttt{null} for missing or non-applicable information, and avoid hallucination.

The extraction prompt began with the following instruction:

\begin{verbatim}
You are a domain expert in fuel-cell electrocatalysis and data engineering.
Your task is to extract a COMPACT set of common, high-value features from an
experimental paper about high-entropy alloy or multi-principal-element alloy
catalysts for fuel-cell-related reactions.
Return a STRICTLY valid JSON object with EXACTLY the keys defined below.
If a feature is not mentioned or not applicable, strictly output null.
Do not hallucinate data.
\end{verbatim}

The fixed JSON schema used by the Agent is listed in Table~\ref{tab:json_schema}. After parsing the model response, the script appended bibliographic metadata from the retrieval record, including paper title, DOI, publication year and journal.

\begin{longtable}{p{0.30\textwidth}p{0.58\textwidth}}
\caption{Fixed feature schema required in the structured extraction prompt.}
\label{tab:json_schema}\\
\toprule
JSON key & Definition used in the prompt \\
\midrule
\endfirsthead
\toprule
JSON key & Definition used in the prompt \\
\midrule
\endhead
\texttt{Catalyst\_Composition} & Catalyst formula or name, such as \texttt{PtPdRuRhIr/C} or \texttt{PtPdNiCoFe nanoparticles}. \\
\texttt{Element\_List} & Metallic elements in the HEA or multimetallic alloy. \\
\texttt{Number\_of\_Elements} & Count of metallic elements in the catalyst alloy. \\
\texttt{Atomic\_Ratio} & Nominal or measured atomic ratio when reported. \\
\texttt{Support} & Catalyst support, such as carbon black, Vulcan XC-72, CNT or graphene; \texttt{null} for unsupported catalysts when appropriate. \\
\texttt{Particle\_Size\_nm} & Average particle size in nanometres when reported. \\
\texttt{Crystal\_Structure} & Phase or structural descriptor, such as fcc, bcc, hcp, amorphous or single-phase fcc. \\
\texttt{Synthesis\_Method} & Synthesis route, such as solvothermal synthesis, wet-chemical reduction, impregnation--reduction or annealing. \\
\texttt{Reaction\_Type} & Reaction class, restricted to ORR, HOR, MOR, EOR, FAOR, AOR, fuel-cell/MEA or another fuel-cell-relevant reaction. \\
\texttt{Test\_Mode} & Electrochemical test format, such as RDE half-cell, MEA single cell or three-electrode testing. \\
\texttt{Electrolyte} & Electrolyte identity and concentration when reported. \\
\texttt{ECSA\_m2\_g} & Electrochemically active surface area, preferably with unit. \\
\texttt{Half\_Wave\_Potential\_V} & ORR half-wave potential, including reference scale if stated. \\
\texttt{Mass\_Activity} & Mass activity with unit and potential when stated. \\
\texttt{Specific\_Activity} & Specific activity with unit and potential when stated. \\
\texttt{Power\_Density} & Fuel-cell peak or maximum power density when reported. \\
\texttt{Activity\_Retention} & Durability result, such as activity, ECSA or current retention after cycles or hours. \\
\texttt{Benchmark\_Catalyst} & Comparator catalyst, such as commercial Pt/C, when reported. \\
\texttt{Key\_Finding} & One short sentence summarising the most relevant design--performance conclusion from the paper. \\
\bottomrule
\end{longtable}

\subsection*{S3.6. JSON parsing, metadata attachment and data persistence}

The extraction script searched the model response for a JSON object, parsed it with a standard JSON parser and discarded responses that could not be parsed. This design prevented free-text explanations or partially structured answers from entering the dataset. For successfully parsed records, the script attached \texttt{Paper\_Title}, \texttt{DOI}, \texttt{Publication\_Year} and \texttt{Journal} from the retrieval metadata. Records were appended to a comma-separated table with UTF-8 encoding. The header was written only when the output file did not already exist, allowing the extraction run to accumulate records incrementally.

This protocol provided four safeguards against unsupported extraction: binary relevance decisions before feature extraction, a fixed compact schema, explicit \texttt{null} output for missing fields and manual audit of a random subset of final records. The open repository reported in the main manuscript contains the final dataset and intermediate Agent records, including generated query entries and candidate-paper metadata, so that the search and extraction path can be inspected beyond the final curated table.

\end{document}
