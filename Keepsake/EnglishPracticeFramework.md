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


1. SOUND & SPELLING
   1.1 The Sound System
      1.1.1 Consonant Sounds
      1.1.2 Vowel Sounds
      1.1.3 Word Stress Rules
      1.1.4 Sentence Stress: Content and Function Words
      1.1.5 Weak Forms
      1.1.6 Intonation: Falling, Rising, Fall-Rise
      1.1.7 Connected Speech: Linking, Elision, Assimilation, Intrusion
   1.2 The Writing System
      1.2.1 The Alphabet: Letters and Letter Names
      1.2.2 Regular Spelling-Sound Correspondences
      1.2.3 Position-Dependent Spellings
      1.2.4 Silent Letters
      1.2.5 Punctuation: Terminal Marks
      1.2.6 Punctuation: Internal Marks
      1.2.7 Punctuation: Grouping Marks
      1.2.8 The Apostrophe: Contraction and Possession
      1.2.9 Capitalization Rules

2. WORD BUILDING
   2.1 Morphemes: Free and Bound
   2.2 Inflection
      2.2.1 Noun Plural Formation
      2.2.2 Irregular Plurals
      2.2.3 Possessive Formation
      2.2.4 Verb Inflection: Tense and Agreement
      2.2.5 Adjective and Adverb Comparison
   2.3 Derivation
      2.3.1 Prefixation
      2.3.2 Suffixation and Word Class Change
   2.4 Compounding
   2.5 Conversion

3. NOUNS & DETERMINATION
   3.1 Noun Types
      3.1.1 Common and Proper Nouns
      3.1.2 Countable and Uncountable Nouns
      3.1.3 Collective Nouns
   3.2 Determiners
      3.2.1 Articles: Definite, Indefinite, Zero
      3.2.2 Article Usage Rules
      3.2.3 Demonstratives
      3.2.4 Possessive Determiners
      3.2.5 Quantifiers with Count Nouns
      3.2.6 Quantifiers with Non-Count Nouns
      3.2.7 Quantifiers with Both Types
      3.2.8 Numerals: Cardinal and Ordinal
      3.2.9 Interrogative Determiners
   3.3 Pronouns
      3.3.1 Personal Pronouns: Subject and Object Forms
      3.3.2 Possessive Pronouns
      3.3.3 Reflexive Pronouns
      3.3.4 Reciprocal Pronouns
      3.3.5 Relative Pronouns
      3.3.6 Interrogative Pronouns
      3.3.7 Indefinite Pronouns
      3.3.8 Impersonal It
   3.4 The Genitive
      3.4.1 Saxon Genitive
      3.4.2 Of-Genitive
      3.4.3 Double Genitive
      3.4.4 Choice and Usage
   3.5 Noun Phrase Structure
      3.5.1 Premodification
      3.5.2 Head Noun
      3.5.3 Postmodification
      3.5.4 Apposition

4. VERBS & VERB PATTERNS
   4.1 Verb Types
      4.1.1 Transitive Verbs
      4.1.2 Intransitive Verbs
      4.1.3 Linking Verbs
      4.1.4 Ditransitive Verbs
      4.1.5 Complex Transitive Verbs
   4.2 Auxiliary Verbs
      4.2.1 Primary Auxiliaries: Be, Have, Do
      4.2.2 Pure Modal Auxiliaries
      4.2.3 Semi-Modals
   4.3 Tense
      4.3.1 Present Simple and Present Continuous
      4.3.2 Past Simple and Past Continuous
      4.3.3 Future Expression: Will and Be Going To
      4.3.4 Future in the Past
      4.3.5 Past Habits: Used To and Would
   4.4 Aspect
      4.4.1 Simple Aspect
      4.4.2 Progressive Aspect
      4.4.3 Perfect Aspect
      4.4.4 Perfect Progressive Aspect
      4.4.5 Stative and Dynamic Verbs
   4.5 Voice
      4.5.1 Active Voice
      4.5.2 Passive Voice Formation
      4.5.3 Passive with Modal Auxiliaries
      4.5.4 Get-Passive
      4.5.5 The Passive Agent
   4.6 Modality
      4.6.1 Ability and Possibility
      4.6.2 Permission and Prohibition
      4.6.3 Obligation and Necessity
      4.6.4 Advice and Recommendation
      4.6.5 Deduction and Certainty
      4.6.6 Offers, Requests, and Willingness
   4.7 Non-Finite Forms
      4.7.1 Bare Infinitive
      4.7.2 To-Infinitive
      4.7.3 Gerund
      4.7.4 Present Participle
      4.7.5 Past Participle
   4.8 Verb Complementation
      4.8.1 Verb + Infinitive
      4.8.2 Verb + Gerund
      4.8.3 Verb + Object + Infinitive
      4.8.4 Verb + Preposition + Gerund
      4.8.5 Verb + Object + Bare Infinitive
      4.8.6 Phrasal Verbs
      4.8.7 Causative Constructions
      4.8.8 Be Used To and Get Used To
   4.9 The Subjunctive

5. MODIFIERS & WORD ORDER
   5.1 Adjectives
      5.1.1 Attributive and Predicative Position
      5.1.2 Adjective Order Rules
      5.1.3 Participial Adjectives
      5.1.4 Adjective and Preposition Collocations
   5.2 Comparison
      5.2.1 Comparative Formation
      5.2.2 Superlative Formation
      5.2.3 Irregular Comparison
      5.2.4 Comparative Constructions
      5.2.5 Correlative Comparison: The...The
   5.3 Adverbs
      5.3.1 Manner Adverbs
      5.3.2 Time Adverbs
      5.3.3 Place Adverbs
      5.3.4 Frequency Adverbs
      5.3.5 Degree Adverbs
      5.3.6 Sentence Adverbs
      5.3.7 Adverb Position: Initial, Medial, End
   5.4 Degree Modifiers
      5.4.1 So and Such
      5.4.2 Too and Enough
      5.4.3 Very and Much
   5.5 Prepositions
      5.5.1 Prepositions of Place
      5.5.2 Prepositions of Time
      5.5.3 Prepositions of Movement
      5.5.4 Dependent Prepositions
   5.6 Prepositional Phrases
      5.6.1 Adverbial Function
      5.6.2 Adjectival Function

6. THE SIMPLE SENTENCE
   6.1 Clause Elements
      6.1.1 Subject
      6.1.2 Verb
      6.1.3 Direct Object
      6.1.4 Indirect Object
      6.1.5 Subject Complement
      6.1.6 Object Complement
      6.1.7 Adverbial
   6.2 Clause Patterns
      6.2.1 SV
      6.2.2 SVO
      6.2.3 SVC
      6.2.4 SVA
      6.2.5 SVOO
      6.2.6 SVOC
      6.2.7 SVOA
   6.3 Concord
      6.3.1 Subject-Verb Agreement
      6.3.2 Pronoun-Antecedent Agreement
   6.4 Negation
      6.4.1 Negation with Auxiliaries
      6.4.2 Do-Support
      6.4.3 Negative Contractions
      6.4.4 Negative Polarity Items
   6.5 Interrogatives
      6.5.1 Yes/No Questions
      6.5.2 Wh-Questions
      6.5.3 Subject Questions
      6.5.4 Question Tags
      6.5.5 Alternative Questions
      6.5.6 Short Responses
   6.6 Imperatives
      6.6.1 Affirmative Imperatives
      6.6.2 Negative Imperatives
      6.6.3 Imperatives with Explicit Subject
   6.7 Exclamatives
   6.8 Existential Constructions

7. COMPLEX SENTENCES
   7.1 Coordination
      7.1.1 Coordinating Conjunctions
      7.1.2 Correlative Conjunctions
      7.1.3 Punctuation in Coordination
   7.2 Nominal Clauses
      7.2.1 That-Clauses
      7.2.2 Wh-Nominal Clauses
      7.2.3 If and Whether Clauses
      7.2.4 Extraposition
   7.3 Relative Clauses
      7.3.1 Defining Relative Clauses
      7.3.2 Non-Defining Relative Clauses
      7.3.3 Relative Pronoun Selection
      7.3.4 Omission of Relative Pronoun
      7.3.5 Reduced Relative Clauses
   7.4 Adverbial Clauses
      7.4.1 Time Clauses
      7.4.2 Concessive Clauses
      7.4.3 Contrast Clauses
      7.4.4 Purpose Clauses
      7.4.5 Reason Clauses
      7.4.6 Result Clauses
      7.4.7 Manner Clauses
      7.4.8 Place Clauses
      7.4.9 Comparison Clauses
   7.5 Non-Finite Clauses
      7.5.1 Infinitive Clauses
      7.5.2 Gerund Clauses
      7.5.3 Participial Clauses
   7.6 Conditionals
      7.6.1 Zero Conditional
      7.6.2 First Conditional
      7.6.3 Second Conditional
      7.6.4 Third Conditional
      7.6.5 Mixed Conditionals
      7.6.6 Inverted Conditionals
      7.6.7 Unless and Provided That
      7.6.8 Implied Conditionals
   7.7 Reported Speech
      7.7.1 Reporting Statements
      7.7.2 Reporting Questions
      7.7.3 Reporting Commands
      7.7.4 Tense Backshift
      7.7.5 Pronoun and Time Shifts
      7.7.6 Passive Reporting Structures
   7.8 Wishes and Unreal Past
      7.8.1 Wish and If Only
      7.8.2 Would Rather and Had Better
      7.8.3 It's Time

8. INFORMATION PACKAGING
   8.1 Fronting and Inversion
      8.1.1 Fronting for Emphasis
      8.1.2 Subject-Verb Inversion
      8.1.3 Negative Inversion
      8.1.4 Conditional Inversion
      8.1.5 Additive Inversion: So and Neither
   8.2 Cleft Sentences
      8.2.1 It-Clefts
      8.2.2 Wh-Clefts
   8.3 Emphasis
      8.3.1 Emphatic Do
      8.3.2 Reflexive Emphasis
   8.4 Ellipsis
      8.4.1 Verb Phrase Ellipsis
      8.4.2 Gapping
   8.5 Information Structure
      8.5.1 Given and New Information
      8.5.2 Theme and Rheme
      8.5.3 End Weight

9. DISCOURSE & PRAGMATICS
   9.1 Cohesion
      9.1.1 Reference: Anaphoric, Cataphoric, Exophoric
      9.1.2 Substitution
      9.1.3 Ellipsis
      9.1.4 Conjunction and Discourse Markers
      9.1.5 Lexical Cohesion
   9.2 Text Organization
      9.2.1 The Paragraph
      9.2.2 Topic Sentence
      9.2.3 Unity and Coherence
   9.3 Register
      9.3.1 Formal Register
      9.3.2 Neutral Register
      9.3.3 Informal Register
   9.4 Speech Functions
      9.4.1 Making Requests
      9.4.2 Giving Advice
      9.4.3 Making Offers
      9.4.4 Giving Opinions
      9.4.5 Apologizing
      9.4.6 Complaining
   9.5 Politeness
      9.5.1 Direct and Indirect Strategies
      9.5.2 Hedging
   9.6 Deixis
      9.6.1 Person Deixis
      9.6.2 Time Deixis
      9.6.3 Place Deixis
      9.6.4 Discourse Deixis
