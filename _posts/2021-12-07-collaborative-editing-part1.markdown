---
layout: post
title:  "A Casual Introduction to Collborative Rich-Text Editing on the Web - Part 1: Editors"
date:   2021-12-07 14:29:25 -0500
categories: article
---

I'm going to go into some of my limited experience learning about and developing web-based collaborative editing software. Hopefully this can act as another data point for anyone considering a path for their's. This is part one in a two part series. The second will detail the algorithms and data structures underpinning many of the collaborative experiences you've likely had or will have. 

# [Quill](https://quilljs.com/)
Probably the most prominent and widely used open source editor on the web right now. You can see it embedded into Slack (as part of their 2019 WYSIWYG Editor update), LinkedIn, and Salesforce. It was first developed by Justin Chen in the early 2010's (probably 2014) who's currently CEO at [Slab](https://slab.com/). I recommend checking the platform out as its a good demonstration of what you can pull off when you understand the editor. 

Libraries for [React](https://github.com/zenoamaro/react-quill), [Vue](https://vueup.github.io/vue-quill/), and [Angular](https://github.com/KillerCodeMonkey/ngx-quill) exist and receive regular or semi-regular updates. It's important to note that you don't have to rely on these options, if you need to target a more recent of Quill you can roll your own - just use those libraries as an example. I recommend these libraries because they simplify the integration process and add some extra features you might find useful. 

There's also an awesome [editor binding](https://docs.yjs.dev/ecosystem/editor-bindings/quill) with [Yjs](https://yjs.dev), which makes collaboration plug and play. We'll talk about CRDTs, the technology that backs Yjs, in part 2. 

**Personal Thoughts:** This is a fantastic choice if you need something that just works. The premade themes look great and most of the text formatting options you'd expect are built in. It's why those aforementioned companies went with Quill. However, it leaves a lot to be desired when it comes to extensibility. Quill uses blots and modules to build extra functionality which is how the add-ons on the [Awesome List](https://github.com/quilljs/awesome-quill) are implemented. These are certainly useful but when it comes to advanced use cases like rendering React components within the editor, this system can be cumbersome. You'll have to handle edge cases yourself and there isn't a lot of high-quality documentation about it. For example, the guide demonstrating how to build your own [Medium clone](https://quilljs.com/guides/cloning-medium-with-parchment/) relies on low-level DOM manipulation. If a Notion or Nuclino-style interface is your end goal (as it was for me) I'd look elsewhere. 

# [Prosemirror](https://prosemirror.net/)
It's very important to emphasize that Prosemirror is not as much an editor as it is a toolkit for building editors. The example shown in the site is very functional but not very pretty and that's by design. Developed by [Marijn Haverbe](https://marijnhaverbeke.nl/) (yes the [CodeMirror](https://codemirror.net/) person), PM sees usage at Atlassian, Asana, and Desmos. 

There are a couple of framework libraries but none I can directly recommend. Unlike the options I presented for Quill there aren't definitive choices for PM. If you want to use PM, you're better off rolling your own. If you're on React you could look at the [Atlaskit editor](https://atlaskit.atlassian.com/packages/editor/editor-core). By the way, that's another point in PM's favor, you can see how a large company is leveraging it for an advanced use case. There's some good examples in there about how to render React components within PM. 

Also has an editor binding with Yjs. Prosemirror also has an extension for collaborative editing that uses a quasi-CRDT, quasi-OT route. Haverbeke describes it as almost like real-time Git in this [blog post](https://marijnhaverbeke.nl/blog/collaborative-editing.html). 

**Personal Thoughts:** I don't have a lot to say here due to my limited usage. ProseMirror is extraordinarly powerful but also very low-level. If you need complete control over your editor and its behavior I'd recommend it. For everyone else, I think TipTap (built on top of PM) provides the both the flexibility of PM and the ease of Quill. 

# [TipTap](https://tiptap.dev/)
I love this editor. I know I'm supposed to reserve my personal thoughts for the end but I cannot wait to recommend this to you. The library, the community, and the documentation is excellent. Okay, maybe not that last one but the team behind it, [Uberdosis](https://ueberdosis.io/), is working on it. While they were primarily invested in Vue ecosystem, they've since branched out into a natively supported React integration. For Angular, you can use this [third party package](https://github.com/sibiraj-s/ngx-tiptap) (caution: haven't vetted this myself). The extensions system frees you from the shackles of PM API and there's built-in support for rendering React components in the editor. 

Yet again, Yjs has a binding you can use for immediate collaborative editing. I was interested in seeing if the original PM collaboration extension could be retrofitted for this editor but I haven't gotten around to trying it. As we'll talk about in part 2, there are tradeoffs between the CRDT approach Yjs uses versus the OT approach that Google Docs uses. Let me know if you've been able to pull it off. 

---

That's it for now. This is a work in progress and will be updated anytime I try out a new editor or figure out something interesting about one of these. Also check out Justin Hunter's [article](https://medium.com/the-lead/why-we-moved-from-quill-to-slate-94f42aa54fec) on why he moved from Quill to Slate. I didn't get to use [Graphite](https://graphitedocs.com) when I first heard about it but from what I've read I'm sad to see its gone. Hopefully, since its open source, someone else can carry the torch. 