+++
title= "How GitHub is losing the AI Code Battle"
date= "2025-02-07"
tags= ["dev", "ai", "github", "copilot", "cursor", "codeium"]
+++

First, I have a lot of respect for GitHub Copilot. I mean, it started the whole wave of the AI coding revolution with its auto-completion and code suggestions. I don't think I've recommended any tool as much as Copilot. In the initial years, if you met me, you probably got a Copilot recommendation about how awesome it was.

But, except for Auto Complete, GitHub seems to have fumbled everything else. I think their missteps can be boiled down to these:

## 1. Copilot Chat Overload

Chatting with codebases has to be the most overhyped thing ever. I'm sure it helps in some cases, but is it as useful as it's made out to be everywhere? It's also now a solved problem. But most of GitHub's work in the last year has been around putting Copilot Chat everywhere i.e., website, pull requests, codebases, etc. I mean, now you have a chat box on every page in GitHub. The whole promise of AI is to *do* things, not just *chat* about them. Having one place where you can chat is good, but everywhere? We already have ChatGPT, Claude, etc., for that.

Even on the GitHub website, Copilot Chat doesn't indicate which AI model it's using for responses (I'm not too optimistic about them using the best model), and there's no model selection capability. I usually find it more effective to copy and paste the context into personal ChatGPT or Claude (This way I can select the best model (including reasoning) for the task).

## 2. Wrong Agentic Abstraction

The initial agent abstraction from GitHub i.e., Copilot Workspace was inside... GitHub. While it makes sense for the company in theory, at a user level, it totally doesn't make sense. Code is written in the editor, and that's where users want to interact with it. No AI-generated code (at least right now) can directly move to review without going through iterations i.e., both prompt/chat and direct editing. The GitHub iteration loop is way too outside the code editing workflow. It's the editor that matters.

This is where Cursor got many things right. It gave the power to the user directly inside their editor with Composer. The iteration loop is crazy fast. It's like magic. 

And I also don't think developers are ready to *show/acknowledge* that their code is written entirely by AI (like in the workspace). They are okay to show that they took help from AI, but it behaviorally does not make sense yet. This is the same problem I see with Devin too.

This essentially led to Cursor becoming the [fastest $100M ARR company in history](https://spearhead.so/cursor-by-anysphere-the-fastest-growing-saas-product-ever/).

And the funny part is, GitHub owns VSCode too. They could've totally built what Cursor did. It was obvious from where they started i.e auto-completion and code suggestions.

(Just this week, [GitHub launched Agent Mode right inside the editor](https://github.blog/news-insights/product-news/github-copilot-the-agent-awakens/) in *beta* 😅 but the vibes seem to have been *set again*. Also, I don't understand why they have three different things
i.e Chat, Edits and Agent mode and expect the user to do the selection part.)

## 3. Nothing for (Self-hosted) Enterprises

Self-hosted source control is a huge thing at many big enterprises, right? GitLab, Bitbucket, etc., are doing great in this space, and GitHub *failed* to provide a Copilot product that completely (including the model) runs in the *company's environment*. A lot of companies who are already using self-hosted source control would want a similar experience even for Code Generation.

This gave rise to [another of the fastest-growing companies right now i.e., Codeium](https://www.businesswire.com/news/home/20240829623867/en/Codeium-Reaches-1.25B-Valuation-with-150M-Series-C-Funding-Led-by-General-Catalyst), who seem to be doing well with big customers like JPMorgan, etc by providing a Copilot product that [runs **entirely** in the company's environment](https://codeium.com/enterprise). A lot of enterprises seem to be wanting this, and Codeium delivered.

(Now Codeium is also competing with Cursor with their product called [Windsurf](https://codeium.com/windsurf), and people seem to be liking it)

## 4. Betting only on OpenAI Models

I guess being owned by Microsoft has its problems too (who would've thought). They completely focused on OpenAI models, while Claude Sonnet was **literally killing it** with code. It's hard to overstate how many leaps and bounds better Sonnet is compared to OpenAI GPT models. (The O1 models could now probably compete, but it's kind of too late for that as the *vibes* have been set)

This meant Copilot was essentially quite bad, and users were starting to use Claude directly and tools like Cody more, as they wanted Sonnet for anything code-related.

I personally found Claude (even while being outside the editor and getting limited manual copy-pasted context) to be quite better than Copilot for many of my problems.

This gave rise to so many similar offerings like [Cody](https://sourcegraph.com/cody), etc who were able to provide the user with an option to choose the model they want (including Sonnet).

Now they have added support for other models (but they are still called *beta* 😅 and are not the defaults). I'm not sure how users would find this appealing, knowing that Copilot has defaulted to the less effective models.

## Conclusion

It's fascinating how one company's strategic missteps became the foundation for multiple startups' product-market fit.

I'm not saying GitHub Copilot is completely failing. It has [done quite well with $300M ARR as of July 2024](https://x.com/ericabrescia/status/1818435925640364402). However, considering their resources with Microsoft, VSCode, and other advantages, they could have achieved much more. It's also nice to see startups innovate and give GitHub a tough time in their own game.

I still consider the AI Code Battle to be still in infancy, and we never know how it's going to play out.
