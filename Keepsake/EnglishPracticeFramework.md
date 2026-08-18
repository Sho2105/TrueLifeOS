<style>
        :root {
            --yuki: #f8f5ee; --kami: #eee8db; --kasumi: #cfc8bc;
            --hai: #8d887d; --sumi: #1a1918; --shu: #923e35;
            --ruri: #7d8d9b; --moegi: #7e866f; --ougon: #b39d77;
            --u: 5px;
        }
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Noto Sans', 'Helvetica Neue', Arial, sans-serif;
            font-size: 10px; line-height: 1.38; color: var(--sumi);
            background: var(--yuki); padding: var(--u);
            letter-spacing: -0.01em; word-spacing: -0.02em;
        }
        header, footer {
            width: 100%; background: var(--kami);
            padding: calc(var(--u)*2) var(--u);
            margin-bottom: var(--u); break-inside: avoid;
        }
        footer { margin-top: var(--u); margin-bottom: 0; }
        #masonry { display: flex; gap: 0; align-items: flex-start; }
        .col { flex: 1; display: flex; flex-direction: column; gap: 0; min-width: 0; }
        .col + .col { margin-left: 0; }
        .block {
            padding: calc(var(--u)*0.8) var(--u);
            background: var(--kami);
            margin-bottom: var(--u);
            break-inside: avoid; page-break-inside: avoid;
        }
        .block:nth-child(even) { background: var(--yuki); }
        h1, h2, h3, h4, h5, h6 {
            font-weight: 600; line-height: 1.1;
            margin: 0.15rem 0 0.05rem; color: var(--sumi);
        }
        h1 { font-size: 1.55em; margin-bottom: 0.25rem; }
        h2 { font-size: 1.25em; } h3 { font-size: 1.1em; } h4 { font-size: 1em; }
        h5 { font-size: 0.95em; color: var(--hai); }
        h6 { font-size: 0.9em; color: var(--kasumi); text-transform: uppercase; letter-spacing: 0.04em; }
        p { margin: 0.08rem 0; text-indent: 0.6em; }
        p:first-of-type, h1+p, h2+p, h3+p, h4+p, h5+p, h6+p, hr+p, blockquote+p { text-indent: 0; }
        hr { border: none; height: 1px; background: var(--kasumi); margin: 0.35rem 0; }
        ul, ol { margin: 0.05rem 0 0.05rem 1.1em; }
        li { margin: 0.02rem 0; } ul ul, ol ol, ul ol, ol ul { margin-left: 0.9em; }
        dl { display: grid; grid-template-columns: auto 1fr; gap: 0.15rem 0.4rem; margin: 0.1rem 0; }
        dt { font-weight: 600; color: var(--hai); font-size: 0.9em; text-transform: uppercase; letter-spacing: 0.03em; }
        dd { margin: 0; }
        table { width: 100%; border-collapse: collapse; margin: 0.2rem 0; font-size: 0.95em; }
        caption { font-weight: 600; text-align: left; font-size: 0.85em; text-transform: uppercase; letter-spacing: 0.05em; color: var(--hai); margin-bottom: 0.1rem; }
        thead th { text-align: left; font-weight: 600; padding: 0.1rem 0.3rem 0.08rem; border-bottom: 1.5px solid var(--sumi); font-size: 0.85em; text-transform: uppercase; letter-spacing: 0.04em; }
        tbody td { padding: 0.06rem 0.3rem; border-bottom: 0.5px solid var(--kasumi); vertical-align: top; }
        tbody tr:nth-child(even) td { background: var(--kami); }
        tfoot td { padding: 0.08rem 0.3rem; font-weight: 600; font-size: 0.85em; color: var(--hai); border-top: 1px solid var(--kasumi); }
        blockquote { margin: 0.15rem 0; padding: 0.15rem 0.5rem; background: var(--kasumi); font-style: italic; color: var(--hai); }
        blockquote p { text-indent: 0; margin: 0.04rem 0; }
        pre { margin: 0.15rem 0; padding: 0.25rem 0.4rem; background: var(--kami); font-family: 'SF Mono', 'Noto Sans Mono', 'Consolas', monospace; font-size: 0.95em; line-height: 1.25; overflow-x: auto; white-space: pre-wrap; word-break: break-all; }
        code { font-family: 'SF Mono', 'Noto Sans Mono', 'Consolas', monospace; font-size: 0.95em; background: var(--kami); padding: 0.02em 0.25em; }
        pre code { background: none; padding: 0; }
        details { margin: 0.12rem 0; padding: 0.1rem 0.3rem; background: var(--kami); cursor: pointer; }
        summary { font-weight: 600; font-size: 0.9em; text-transform: uppercase; letter-spacing: 0.04em; color: var(--hai); padding: 0.05rem 0; }
        details[open] { padding-bottom: 0.2rem; }
        figure { margin: 0.2rem 0; text-align: center; }
        figcaption { font-size: 0.85em; color: var(--kasumi); font-style: italic; margin-top: 0.05rem; }
        mark { background: var(--ougon); color: var(--sumi); padding: 0.02em 0.2em; }
        del { color: var(--kasumi); text-decoration: line-through; text-decoration-color: var(--shu); }
        ins { text-decoration: underline; text-decoration-color: var(--sumi); text-underline-offset: 0.15em; }
        u { text-decoration: underline; text-underline-offset: 0.12em; text-decoration-color: var(--hai); }
        a { color: var(--shu); text-decoration: underline; text-underline-offset: 0.12em; }
        small { font-size: 0.85em; color: var(--hai); }
        strong { font-weight: 700; color: var(--sumi); }
        em { font-style: italic; color: var(--hai); }
        abbr[title] { text-decoration: underline dotted; text-underline-offset: 0.15em; cursor: help; color: var(--hai); }
        q { font-style: italic; color: var(--hai); }
        q::before { content: '「'; color: var(--kasumi); }
        q::after { content: '」'; color: var(--kasumi); }
        cite { font-style: italic; color: var(--kasumi); }
        kbd { font-family: 'SF Mono', 'Noto Sans Mono', 'Consolas', monospace; font-size: 0.9em; background: var(--kasumi); padding: 0.04em 0.35em; border-radius: 2px; }
        samp { font-family: 'SF Mono', 'Noto Sans Mono', 'Consolas', monospace; font-size: 0.95em; }
        var { font-style: italic; color: var(--hai); }
        ruby { ruby-align: center; } rt { font-size: 0.6em; color: var(--kasumi); } rp { font-size: 0.6em; color: var(--kasumi); }
        address { font-style: normal; font-size: 0.9em; color: var(--hai); margin: 0.1rem 0; }
        time { color: var(--hai); font-variant-numeric: tabular-nums; }
        sub, sup { font-size: 0.75em; line-height: 0; position: relative; } sup { top: -0.45em; } sub { bottom: -0.3em; }

        @media print {
            body { font-size: 9.5px; line-height: 1.32; padding: var(--u); background: #fff; color: #000; }
            :root { --yuki: #fff; --kami: #f5f5f5; --kasumi: #d0d0d0; --hai: #555; --sumi: #000; --shu: #8b1a1a; --ougon: #d4c098; --ruri: #5d6d7e; --moegi: #5d6e52; }
            .block { background: var(--kami); } .block:nth-child(even) { background: #fff; }
            pre, code, blockquote, details, tbody tr:nth-child(even) td { background: #f8f8f8; }
            thead th { border-bottom-color: #000; } tbody td { border-bottom-color: #ddd; } hr { background: #ddd; }
            @page { size: A4; margin: 0.5cm; }
        }
        @media screen and (max-width: 600px) {
            #masonry { flex-direction: column; } .col { width: 100%; flex: none; } dl { grid-template-columns: 1fr; }
        }
</style>

Input:
Generate a CEFR [A1/A2/B1/B2/C1/C2] daily practice test.
Root: [word]
Theme: [short theme]
Extra instructions: [e.g., “include all 9 parts of speech, diverse tenses, no forced vocab”]
Output ONLY the test sheet below — no explanations, no extra text:
Name: ________________________________________   Date: DD/MM/YY
1. VOCABULARY (ROOT FAMILY)
Root: [word]
/[IPA: p b t d k g f v s z m n l r w h j θ ð ʃ ʒ tʃ dʒ ŋ iː ɪ e æ ɑː ɒ ɔː ʊ uː ʌ ɜː ə eɪ aɪ ɔɪ aʊ]/
POS : N | Adj | Det | V | Adv | Pro | Pre | Con | Int
Family (underline affix):
  [prefix]root   /IPA/   POS   definition
  root[suffix]   /IPA/   POS   definition
  [prefix]root   /IPA/   POS   definition
Phrasal verbs (only if they fit the theme naturally):
  [phrasal]   definition   e.g. [example sentence]
...
2. READING
...
3. WRITING
...
4. LISTENING
...
5. SPEAKING
....

=== VOCABULARY (root family) ===
Root: [word]
/[IPA: p b t d k g f v s z m n l r w h j θ ð ʃ ʒ tʃ dʒ ŋ iː ɪ e æ ɑː ɒ ɔː ʊ uː ʌ ɜː ə eɪ aɪ ɔɪ aʊ]/
POS : N | Adj | Det | V | Adv | Pro | Pre | Con | Int
Family (underline affix):
  [prefix]root   /IPA/   POS   definition
  root[suffix]   /IPA/   POS   definition
  [prefix]root   /IPA/   POS   definition
Phrasal verbs (only if they fit the theme naturally):
  [phrasal]   definition   e.g. [example sentence]

=== READING ===
Text: [a passage on the theme. Use vocabulary family members only if they appear naturally. Do not force.]
Length & complexity:
  A1: 30‑50 words; simple sentences, basic conjunctions.
  A2: 80‑100 words; compound sentences, everyday lexis.
  B1: 120‑150 words; simple paragraphs, some dependent clauses.
  B2: 180‑250 words; clear argument, linking words of contrast/cause.
  C1: 250‑350 words; abstract ideas, tone, figurative language.
  C2: 350‑450 words; dense academic/literary style, complex syntax.

Questions (answer freely, in full sentences):
A1: 1. Find and copy these words/phrases from the text: [2‑3 items].  2. Answer: [explicit detail question].  3. Answer: [another explicit detail question].
A2: 1. Who wrote the text, why, and what is the main message? Explain in a few sentences.  2. Give the changed details (time, place, reason) in your own words.  3. Write a short reply to the message.
B1: 1. Summarise the story in your own words (what happened, feelings, why).  2. What is the writer’s real opinion about the event? Explain what clues you used.  3. Rewrite the last paragraph without changing the meaning.
B2: 1. State the writer’s thesis in one sentence.  2. Summarise the arguments for and against, including the evidence used.  3. Write a paragraph giving your own justified stance.
C1: 1. Explain the central theme and how two stylistic devices (irony, tone, imagery) support it; quote examples.  2. Rewrite the opening paragraph in a sharply different tone.  3. Identify and discuss a weakness in the reasoning or narrative.
C2: 1. Synthesise the core idea with a concept from [discipline]; produce a thesis statement.  2. Critique the author’s use of [rhetorical strategy] and suggest a stronger alternative.  3. Draft an abstract for a paper that places the text in a wider cultural debate.

=== WRITING ===
Open prompt on the daily theme. No word‑count restrictions; the student writes as much as they need.
A1: Fill in the form: Name, Age, Country. Then write a few sentences about what you like doing.
A2: Write a short personal message (thank‑you / invitation / apology) about a specific situation. Use appropriate opening and closing.
B1: Write a personal letter narrating a surprising event. Describe what happened, your feelings, and what it meant to you.
B2: “Some people believe [statement]. Others disagree.” Write an essay presenting both views, your opinion with a concession, and a conclusion.
C1: You conducted a small survey on [theme]. Write a formal report with headings, impersonal tone, hedging, and a description of numerical trends. End with recommendations.
C2: Write a critical review of a [book/film/policy] related to [theme]. Dissect ambition, technique, cultural context. Distinguish intent from achievement and offer a refined evaluation.

=== LISTENING & SPEAKING ===
Teacher reads the script below aloud at appropriate speed (very slow A1 → natural C1/C2). Student listens, then answers orally.

Script (teacher reads from this sheet; student can see these keywords):
A1: [keywords: name, age, country, hobby, favourite food]
    Tasks: 1. Ask the teacher similar questions to find out the same information.  2. Describe the scene from the script in your own words.
A2: [keywords: event, original time, new time, new place, reason for change]
    Tasks: 1. Say what changed and why.  2. Describe a similar past experience of your own in connected sentences.
B1: [keywords: destination, three activities, best moment, one problem]
    Tasks: 1. Summarise the interview in full sentences.  2. Answer two follow‑up questions the teacher will ask about your own experience.
B2: [keywords: Speaker 1 opinion + reasons + example; Speaker 2 opposite opinion + counter‑argument + example; conclusion]
    Tasks: 1. Summarise each speaker’s position and evidence.  2. Present and defend your own balanced view when challenged by the teacher.
C1: [keywords: stated topic, 2‑3 details, one implied criticism, unstated thesis hint]
    Tasks: 1. Reconstruct the implicit thesis and critique.  2. Evaluate the evidence: is it sufficient? Why/why not?  3. Spontaneous debate with the teacher; reformulate and refute.
C2: [keywords: Speaker A position + concession; Speaker B position + rhetorical strategy; Speaker C attempted synthesis]
    Tasks: 1. Reconstruct each speaker’s exact position, noting concessions and strategies.  2. Give a critical synthesis of the exchange.  3. Pivot into unscripted debate; handle nuances and backtracking; synthesise opposing views into a novel perspective.

Section 1:
Text: [a passage on the theme. Use vocabulary family members naturally. Do not force.]
Length & complexity:
  A1: 30‑50 words; simple sentences, basic conjunctions.
  A2: 80‑100 words; compound sentences, everyday lexis.
  B1: 120‑150 words; simple paragraphs, some dependent clauses.
  B2: 180‑250 words; clear argument, linking words of contrast/cause.
  C1: 250‑350 words; abstract ideas, tone, figurative language.
  C2: 350‑450 words; dense academic/literary style, complex syntax.

Questions pool:
- Fill in the blank (part of speech, vocabulary, phrasal verbs, collocations)
- Multiple choice (part of speech, definition, synonyms, antonyms, main idea, inference)
- Rewrite (active/passive, direct/reported, conditional/inversion, relative clauses, causative)
- Word Formation (change base word in brackets to noun, verb, adjective, or adverb)
- Key Word Transformation (rewrite sentence using a mandatory constraint word in 2–6 words)
- Error Correction (find and correct grammar, spelling, or punctuation mistakes in a line)
- Text-Matching / Summary (fill summary gaps using exact words from the passage)
- Short Production (write 20–30 words to justify an opinion or extend the text)


