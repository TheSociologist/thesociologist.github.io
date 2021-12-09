---
layout: post
title:  "A Casual Introduction to Collaborative Rich-Text Editing on the Web - Part 2: Algorithms and Data Structures"
date:   2021-12-09 14:29:25 -0500
categories: article
---

You can check out part 1 [here](/article/2021/12/07/collaborative-editing-part1). 

Collaborative software generally uses either Operational Transform (OT) or Conflict-Free Replicated Data Types (CRDT). There are some platforms that use both, most notably Figma, but for the most part it splits into these two camps. 

# OT
OT has been around for a lot longer which is why you see it in websites like Google Docs, Word's collaborative editing, and just about every collaborative text editor. OT works by looking at the changes that users have made and ensuring they are commited to the document in a predictable manner. The more users there are making changes, the worst OT functions, primarily as a result of its time complexity. This is why Google Docs can't deal with more than 100 users. This is generally not that big of a deal, because c'mon, when are a 100 people gonna edit a document at the same time? However this limit could be an issue with other kinds of collaborative software. Imagine a brainstorming application that provides users with a virtual mind map. Brainstorming could easily involve more than a 100 people (think about a large corporate meeting). 

# CRDT
CRDTs, on the other hand, are much newer with the term being formally described in 2011. Its important to note that collaborative software is not what CRDTs are primarily used for. Rather, most research is spearheaded by their necessity in distributed computing. Horde, helps manage distributed Elixir deployments. Riak, Redis, and CosmosDB all have CRDT implementations you can use for whatever you'd like. When merging problems, you have to transmit the entire data structure to other nodes which can introduce scalability concerns. These don't pop up as quickly as they do with OT but it is a concern.  CRDTs generally come in two flavors, state-based and operation-based which function exactly as their name implies. 

State-based CRDTs are a lot more common (partially because they're easier to implement) but may cause network congestion. You need to transmit the entire data structure to every other node to sync state. Common solutions to this problem involve gossip protocols, rather than trying to have a single node transmit state to all other nodes (like in a fully meshed network), each node transmits to a subset. This increases the delay before every node finishes merging the changes but can improve overall performance. 

Operation-based CRDTs are much more difficult to implement but can reduce congestion. Instead of complete state, they only transmit changes a la OT. However the communication must provide guarantees that all changes are not duplicated or dropped and that they must be sent in causal order. 

The Yjs library as mentioned in the previous article is a fantastic implementation of a CRDT. Its bindings with editors demonstrate its usefulness in collaborative text editing but its also been used for collaborative drawing, 3D visualization, and coding (examples found [here](https://github.com/yjs/yjs-demos/)). 

As a side note, CRDTs are mathematically provable (if you understand lattices from abstract algebra, [knock yourself out](https://crdt.tech/papers.html)).

# CRDTs vs OT
Before we continue, go read [CRDTs are the Future](https://josephg.com/blog/crdts-are-the-future/) and [To OT or CRDT, that is the question](https://www.tiny.cloud/blog/real-time-collaboration-ot-vs-crdt/). If you didn't feel like reading either, the basic gist is this; CRDTS are a lot less messy than OT but they may not be the best at text editing.

Unfortunately, as with most theoretical CS, there is no single best choice, simply tradeoffs. CRDTs being provable is great for distributed systems but mean nothing for usability. While they merge changes in a consistent way, there's no telling if the merge makes human sense. OT, however, can decide to make merges in an understandable way. 

Here's the thing, I don't think that merge conflicts actually appear that often in real life. I mean, how often are you and someone else editing the exact same sentence much less the same word. People generally stay out of each other's way when collaborating, so does this downside really matter? Personally, I'll be using CRDTs for my collaborative applications, primarily because most of the hard work has already been done by the bright people working on Yjs. And when [**hocuspocus**](https://tiptap.dev/hocuspocus/) comes out (hopefully, with everything promised), creating collaborative software development will become the most accessible its ever been. 

---

As a final note, There is kind of a third option. Namely, not handling conflicts at all. Evernote, for example, creates multiple versions of shared documents the user has to manually merge later. This isn't the most user-friendly option but it works if you don't have the time (or patience) for actual implementation. 

Notion has an interesting method for bypassing this issue. Its pages are real-time collaborative but it locks individual blocks from non-editing users.