# Sample Pipeline flowchart - Meme ID: `vbaZHQcZD`

From `--single --debug` on 2026-08-17. Verdict: commentable: **yes**.

Scroll down. Each block is one pipeline segment: caption, debug field, data, then what that step does.

---

## Fetch meme

`single.fetch.request` -> `single.fetch.result`

```json
{
  "memeId": "vbaZHQcZD",
  "type": "pic",
  "source": "mention",
  "link": "https://ifunny.co/picture/vbaZHQcZD",
  "creator": "HeresYourGooberMeal",
  "title": "LOL!!! :)",
  "smiles": 588,
  "comments": 104,
  "views": 30100
}
```
"The bot discovers one page from Featured, Collective, Top of Day, and Mentions. Mentions cut in front of the line."
"Above is a sample of a memes link and metadata, present in that pages data."
"It then rips the meme image link from each meme on every page, and ingests it with `analyze.start`, seen next."

->

## Analyze start

`analyze.start`

```json
{
  "id": "vbaZHQcZD",
  "type": "pic",
  "ifunnyTitleIgnored": "LOL!!! :)",
  "mentionMode": false,
  "skipBlacklist": true,
  "apiOcrText": "Harry Potter\nHarry Potter fans force\nrelocation of undersea cable to avoid\nDobby's 'grave'\nPower cable linking UK and Ireland now passes near Bronze Age human burial remains instead of fictional grave on Welsh beach"
}
```

"The bot begins analysis on the next meme in the queue."

->

## Readiness check

`analyze.spending_lock`

```json
{ "locked": false }
```

"The bot denies analysis, emptying the queue if LLM integrations are erroring or have run out of credits."

->

## Safety check

`safety.check` / `analyze.safety`

```json
{
  "input": "Harry Potter\nHarry Potter fans force\nrelocation of undersea cable to avoid\nDobby's 'grave'\nPower cable linking UK and Ireland now passes near Bronze Age human burial remains instead of fictional grave on Welsh beach",
  "output": {
    "passed": true,
    "verdict": "safe",
    "reason": "No blocklist terms detected in API OCR text.",
    "hits": []
  }
}
```

"Safety runs on API OCR only, before the image is downloaded. Blocklist hits discard the meme immediately."
"Mainly filters for illegal content, not expletives."

->

## Image download

`fetch.download_image`

```json
{
  "id": "vbaZHQcZD-v2-ocr",
  "cached": true,
  "filePath": "cache/images/vbaZHQcZD-v2-ocr.jpg"
}
```

![Downloaded meme](samples/samplememe.jpg)

"The bot downloads the meme JPEG. Before OCR it crops off the black iFunny footer (~22px on this image) so Tesseract does not catch the "iFunny.co" logo text. If it's been pulled before, uses cached image."

->

## Tesseract OCR

`ocr.recognize`

```json
{
  "footerPx": 22,
  "imageSize": { "width": 1080, "height": 1635 },
  "ms": 498,
  "confidence": 84,
  "paragraphs": [
    {
      "text": "Ng py %\n7 iN ’\ni\nHarry Potter\nHarry Potter fans force\nrelocation of £430m\nundersea cable to avoid\nDobby’s ‘grave’\nPower cable linking UK and Ireland now\npasses near Bronze Age human burial\nremains instead of fictional grave on Welsh\nbeach",
      "bbox": { "x0": 29, "y0": 92, "x1": 1026, "y1": 1611 },
      "confidence": 85,
      "lines": [
        { "text": "Ng py %", "bbox": { "x0": 167, "y0": 92, "x1": 561, "y1": 194 }, "confidence": 29 },
        { "text": "7 iN ’", "bbox": { "x0": 284, "y0": 312, "x1": 625, "y1": 426 }, "confidence": 17 },
        { "text": "i", "bbox": { "x0": 1008, "y0": 770, "x1": 1026, "y1": 812 }, "confidence": 96 },
        { "text": "Harry Potter", "bbox": { "x0": 31, "y0": 893, "x1": 336, "y1": 941 }, "confidence": 96 },
        { "text": "Harry Potter fans force", "bbox": { "x0": 32, "y0": 953, "x1": 890, "y1": 1037 }, "confidence": 96 },
        { "text": "relocation of £430m", "bbox": { "x0": 31, "y0": 1049, "x1": 790, "y1": 1126 }, "confidence": 93 },
        { "text": "undersea cable to avoid", "bbox": { "x0": 31, "y0": 1146, "x1": 922, "y1": 1214 }, "confidence": 96 },
        { "text": "Dobby’s ‘grave’", "bbox": { "x0": 32, "y0": 1242, "x1": 586, "y1": 1326 }, "confidence": 84 },
        { "text": "Power cable linking UK and Ireland now", "bbox": { "x0": 31, "y0": 1394, "x1": 944, "y1": 1445 }, "confidence": 96 },
        { "text": "passes near Bronze Age human burial", "bbox": { "x0": 31, "y0": 1453, "x1": 891, "y1": 1504 }, "confidence": 96 },
        { "text": "remains instead of fictional grave on Welsh", "bbox": { "x0": 31, "y0": 1511, "x1": 1015, "y1": 1562 }, "confidence": 96 },
        { "text": "beach", "bbox": { "x0": 29, "y0": 1571, "x1": 169, "y1": 1611 }, "confidence": 96 }
      ]
    }
  ]
}
```

"Tesseract extracts text and the pixel box of every line. Noise (`Ng py %`, `7 iN ’`) is leftover from the photo, and is ignored due to it's low confidence rating."

->

## Spatial clustering

`extract.ocr_tesseract`

```json
{
  "ocrConfidence": 84,
  "groups": [
    {
      "id": "group_1",
      "location": "middle-right",
      "text": "i",
      "x": 1008, "y": 770, "w": 18, "h": 42,
      "chrome": true
    },
    {
      "id": "group_2",
      "location": "bottom",
      "text": "Harry Potter Harry Potter fans force relocation of £430m undersea cable to avoid Dobby’s ‘grave’",
      "x": 31, "y": 893, "w": 891, "h": 433,
      "chrome": false
    },
    {
      "id": "group_3",
      "location": "bottom",
      "text": "Power cable linking UK and Ireland now passes near Bronze Age human burial remains instead of fictional grave on Welsh beach",
      "x": 29, "y": 1394, "w": 986, "h": 217,
      "chrome": false
    }
  ],
  "clusteredText": "Harry Potter Harry Potter fans force relocation of £430m undersea cable to avoid Dobby’s ‘grave’\n\nPower cable linking UK and Ireland now passes near Bronze Age human burial remains instead of fictional grave on Welsh beach"
}
```

"Nearby OCR lines are clustered into spatial groups. Window chrome (useless noise) is marked and dropped. Nearby lines become headline vs body."

->

## Text segments

`extract.result` / `analyze.extract`

```json
{
  "wordCount": 36,
  "ocrConfidence": 84,
  "segments": {
    "masthead": null,
    "headline": "Harry Potter Harry Potter fans force relocation of £430m undersea cable to avoid Dobby’s ‘grave’",
    "body": null,
    "cleaned": "Power cable linking UK and Ireland now passes near Bronze Age human burial remains instead of fictional grave on Welsh beach",
    "hasNewsSignals": false
  }
}
```

"The extract step turns clustered groups into named channels (headline, body, cleaned, URL slug metadata) for later embedding."
"Optional regex filters for common terms used in news outlets boosts probability that meme will pass filtering later."

->

## Soft informational prefilter

`soft_informational.signals` / `analyze.soft_informational`

```json
{
  "claimHits": 0,
  "jokeHits": 0,
  "infoHits": 0,
  "newsLike": false,
  "action": "uncertain",
  "passed": true,
  "reason": "Soft prefilter: ambiguous - pass through to arrange."
}
```

"Cheap keyword/signal check. Obvious jokes can die here. Ambiguous posts pass through instead of being rejected."

->

## Weak embedding gate

`embed.classify` -> `embed.weak_decision` -> `analyze.weak_embed`

```json
{
  "channels": ["headline", "mastheadHeadline", "cleaned", "metadata"],
  "scores": {
    "article_screenshot": 0.284,
    "irony_sarcasm": 0.243,
    "factual_claim": 0.217,
    "gaming_meme": 0.212,
    "causal_opinion": 0.206,
    "map_dataviz": 0.178,
    "reaction_humor": 0.165,
    "engagement_bait": 0.140
  },
  "route": "article_screenshot",
  "action": "pass",
  "reason": "Weak embed prefilter: claim signals + analyze@0.284 - keep for fact-check."
}
```

"An embedding filter is essentially a computationally cheap way of figuring out "if one thing is similar to another type of thing".
"Each text channel is embedded and scored against claim vs humor prototypes. Strong humor dies here. This meme had claim signals, so it continued."
"The threshold values used to determine if it passes or fails were tuned manually against a baseline of 2,000 featured memes."
"Latest 1,000 meme test run scored 97% accuracy, with 30 out of 1,000 memes being commented on when they were nuanced jokes, or being wrongfully denied a comment despite being informational."

->

## Vision structure

`vision_structure.request` -> `vision_structure.response`

```json
{
  "provider": "openai",
  "model": "gpt-4o-mini",
  "synopsis": "The image contains a top caption referencing Harry Potter and a body text explaining the relocation of an undersea cable.",
  "groups": [
    {
      "role": "top_caption",
      "text": [
        "Harry Potter",
        "Harry Potter fans force relocation of £430m undersea cable to avoid Dobby’s ‘grave’"
      ],
      "meaning": "Title and main headline about the relocation of the cable due to fan concerns."
    },
    {
      "role": "body",
      "text": [
        "Power cable linking UK and Ireland now passes near Bronze Age human burial remains instead of fictional grave on Welsh beach"
      ],
      "meaning": "Additional information explaining the reason for the cable's relocation."
    }
  ]
}
```

"A cheap vision model looks at the image plus spatial OCR groups and labels each block (caption vs body vs watermark) without trying to fact-check yet."
"This provides context for text placement, translating our spatial clustering into LLM-consumable context."

->

## Arrange text

`arrange.request` -> `arrange.response` / `analyze.arrange`

```json
{
  "coherent": true,
  "arranged": "Harry Potter fans force the relocation of a £430m undersea cable to avoid Dobby’s ‘grave’. The power cable linking the UK and Ireland now passes near Bronze Age human burial remains instead of a fictional grave on a Welsh beach."
}
```

"OCR is often duplicated and out of order. Arrange rewrites it into one readable paragraph using spatial groups + vision roles."
"This is the step that forms the "meat" of any informational claim being made, if one is being made at all."

->

## Segments after arrange

`analyze.segments_after_arrange`

```json
{
  "headline": "Harry Potter Harry Potter fans force relocation of £430m undersea cable to avoid Dobby’s ‘grave’",
  "body": "The power cable linking the UK and Ireland now passes near Bronze Age human burial remains instead of a fictional grave on a Welsh beach.",
  "cleaned": "Harry Potter fans force the relocation of a £430m undersea cable to avoid Dobby’s ‘grave’. The power cable linking the UK and Ireland now passes near Bronze Age human burial remains instead of a fictional grave on a Welsh beach."
}
```

"Headline stays the OCR cluster. Body/cleaned become the arranged paragraph."
"This step arranges text in a consumable format for the next embeddings filter."

->

## Full embedding gate

`embed.classify` (second pass) / `analyze.full_embed`

```json
{
  "channels": ["headline", "mastheadHeadline", "cleaned", "metadata", "structure"],
  "structure": "The image contains a top caption referencing Harry Potter and a body text explaining the relocation of an undersea cable.",
  "scores": {
    "article_screenshot": 0.304,
    "factual_claim": 0.257,
    "irony_sarcasm": 0.255,
    "gaming_meme": 0.212,
    "causal_opinion": 0.206,
    "map_dataviz": 0.203,
    "reaction_humor": 0.203,
    "engagement_bait": 0.172
  },
  "route": "article_screenshot",
  "shouldAnalyze": true,
  "confidence": "medium",
  "reason": "Semantic match to news/article screenshot claim."
}
```

"Second embedding pass includes the vision synopsis as a `structure` channel.
"The requirements are tighter here, now that text has been arranged into a more understandable format, and more context is provided."
"Most ironic jokes or comparison-style memes die here."

->

## Photo crop for reverse image search

`ris.photo_crop`

```json
{
  "cropped": true,
  "reason": "photo_band",
  "band": { "y0": 4, "y1": 902, "h": 898 },
  "footerPx": 22,
  "image": { "width": 1080, "height": 1635 }
}
```

![Photo band sent to Yandex](samples/sample-photo-crop.jpg)

"If the meme has a large photo region without OCR, that band is cropped and uploaded instead of the full meme (headline text would pollute reverse-image hits)."

->

## Reverse image search

`relatedness.reverse_image`

```json
{
  "risInput": { "skipUrl": true, "localImagePath": "cache/images/vbaZHQcZD-v2-ocr-photo.jpg" },
}
```

"Yandex reverse image search upload helps determine if the meme is nonsense / a joke. It also helps catch illegitimate sources, such as if it pops up from facebook joke accounts."

->

## Relatedness decision

`analyze.relatedness`

```json
{
  "imageDiscarded": true,
  "keepImage": false,
  "reason": "Reverse image found no useful hits; proceeding text-only.",
  "embedding": null,
  "llm": null
}
```

"No useful image matches -> analysis continues on text only. The original photo is not treated as supporting evidence."

->

## Claim extract

`claim_extract.request` -> `claim_extract.response`

```json
{
  "hasVerifiableClaim": true,
  "primaryClaim": "Harry Potter fans forced the relocation of a £430m undersea cable to avoid Dobby’s ‘grave’.",
  "secondaryClaims": [
    "The power cable linking the UK and Ireland now passes near Bronze Age human burial remains instead of a fictional grave on a Welsh beach."
  ]
}
```

"An LLM pulls verifiable claims from arranged text, using vision structure as placement context (caption vs body)."

->

## Verification target

`analyze.verification_target`

```json
{
  "kind": "news_article",
  "isSocialCommentary": false,
  "primaryClaim": "Harry Potter Harry Potter fans force relocation of £430m undersea cable to avoid Dobby’s ‘grave’",
  "attribution": null
}
```

"Picks the research mode. This post looks like a news screenshot, not social commentary."

->

## Claim match

`claim_match.request` -> `claim_match.raw_response` / `analyze.claim_match`

```json
{
  "action": "edit",
  "worthCommenting": true,
  "anecdotalKind": "public_interest",
  "rhetoricalStance": "literal",
  "primaryClaim": "Harry Potter fans influenced the relocation of a £430m undersea cable to avoid Dobby’s ‘grave’.",
  "surfaceClaim": "Harry Potter fans forced the relocation of a £430m undersea cable.",
  "impliedClaims": [
    "The relocation was primarily due to fan concerns about Dobby's grave."
  ],
  "researchAngles": [
    "What evidence exists regarding the influence of fan concerns on the decision to relocate the cable?",
    "What are the implications of the cable's new route passing near Bronze Age burial remains?",
    "What was the process for deciding the cable's original and new routes?"
  ],
  "reason": "The claims about the cable's relocation and the reasons behind it are factual and of public interest, warranting a comment."
}
```

"Claim-match can drop, keep, or edit the extracted claims. Here it softened `forced` -> `influenced` and marked the post worth commenting."

->

## Research start

`research.start`

```json
{
  "researchPath": "lean",
  "queries": [
    "Harry Potter fans influenced the relocation of a £430m undersea cable to avoid Dobby’s ‘grave’.",
    "Harry Potter fans influenced the relocation of a £430m undersea cable to avoid Dobby’s ‘gr false OR debunked OR refuted OR misleading OR cherry-picked"
  ]
}
```

"Lean path: two web searches (claim + a debunk-shaped query). Heavy topics get a fuller research path."

->

## Web search

`research.search_query`

```json
{
  "resultCount": 5,
  "hits": [
    { "source": "The Guardian", "title": "Harry Potter fans force relocation of £430m undersea cable…" },
    { "source": "Firstpost", "title": "Harry Potter fans save Dobby's 'grave' as £430m power cable is rerouted" },
    { "source": "ScreenRant", "title": "Harry Potter Controversy Rallies Fans To Protect Dobby's Gravesite…" },
    { "source": "HeritageDaily", "title": "Harry Potter fans influence change to £430m project…" },
    { "source": "The Mirror", "title": "Harry Potter fans force diversion of £430m undersea cable…" },
    { "source": "The Telegraph", "title": "Harry Potter elf's 'grave' forces diversion of £430m energy project" }
  ]
}
```

"DuckDuckGo snippets are collected. Reddit/anonymous forums are treated as weak sources later."

->

## Fact-check

`research.response` / `analyze.research`

```json
{
  "model": "grok-4-1-fast-reasoning",
  "verdict": "true",
  "recommendation": "amplify",
  "confidence": "high",
  "claimSummary": "Multiple outlets confirm fans' objections led to rerouting the Greenlink cable around Dobby's site at Freshwater West.",
  "closestEvidence": [
    {
      "source": "The Guardian",
      "finding": "fans forced relocation after complaining it would desecrate Dobby's grave"
    },
    {
      "source": "The Telegraph",
      "finding": "hundreds of fans forced diversion of £430m project from the beach"
    }
  ],
  "contextGaps": ["Bronze Age remains not mentioned in any snippet"]
}
```

"A reasoning model reads only the snippets and returns a verdict. Secondary claim about Bronze Age remains was not backed by these snippets, so it becomes a caveat."

->

## Comment

`comment.format` / `analyze.comment`

```json
{
  "depth": 2,
  "comment": "Supported - Multiple outlets confirm fans' objections led to rerouting the Greenlink cable around Dobby's site at Freshwater West. Caveat: Bronze Age remains not mentioned in any snippet. Claim: Harry Potter fans influenced the relocation of a 430m undersea cable to avoid Dobby's 'grave'. (1/2)",
  "replies": [
    "Related: The power cable linking the UK and Ireland now passes near Bronze Age human burial remains instead of a fictional grave on a Welsh beach. Closest evidence: The Guardian: fans forced relocation after complaining it would desecrate Dobby's grave. (2/2)"
  ]
}
```

"This is the comment that would have been queued as a 2-deep thread."

->

## Done

`analyze.done` / `single.report`

```json
{
  "id": "vbaZHQcZD",
  "finalVerdict": "amplify",
  "pipelinePassed": true,
  "commentable": true,
  "primaryClaim": "Harry Potter fans influenced the relocation of a £430m undersea cable to avoid Dobby’s ‘grave’."
}
```

"End of the run. The meme would be eligible to comment in a live loop; this debug pass stopped here."
