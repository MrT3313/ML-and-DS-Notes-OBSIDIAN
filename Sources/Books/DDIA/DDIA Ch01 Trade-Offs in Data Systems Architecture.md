---
note_kind: source
medium: book
chapter: 1
up: "[[DDIA]]"
url: https://github.com/ept/ddia2-references/blob/master/chapter-01-refs.md
aliases:
  - DDIA Chapter 1
  - Trade-Offs in Data Systems Architecture
---

## Thesis

Chapter 1 fixes the axes that the rest of [[DDIA]] places techniques on, and insists in each case that the axis is a trade-off and not a ranking. Three carry it. What the system is for separates a flood of small requests touching records in their current state from a few enormous queries sweeping history for a summary, and that single question, what one request does to the data, forces different storage layouts, different schemas and eventually different machines, and drags a second stack behind it with a job title attached. Who runs the machines is a question about operational skill and the predictability of load before it is a question about technology: your own hardware is usually cheaper if you can already operate the system and can size the load once, and paying a vendor does not remove operations work, it turns capacity planning into financial planning and performance tuning into cost control. One machine or many is where the chapter is most honest, because almost none of its reasons for distributing are about speed: users on separate devices who have to reach each other, surviving a machine that dies, a regulator requiring that particular bytes sit inside a particular country, and against all of it a published measurement where a cluster of more than a hundred cores loses to a single thread. Underneath all three runs the most portable idea in the chapter, which copy of a value is authoritative and which is a rebuildable derivative, and that one rule is what makes a cache, a warehouse, an index and a replica one idea rather than four, because each is allowed to be stale, redundant and deletable precisely because it never gets to win a disagreement. The chapter names no algorithm and settles no question about how to build anything: its content is vocabulary, and it is the vocabulary the rest of the book would be unreadable without.

## Extracted

The axis the whole book is written on:
- [[Data-Intensive Application]]

Which copy of a value wins a disagreement:
- [[System of Record]]
- [[Derived Data]]

Running the business against studying it:
- [[Online Transaction Processing]]
- [[Online Analytical Processing]]
- [[Hybrid Transactional-Analytical Processing]]
- [[Point Query]]
- [[Transaction]]

Where analytical data sits, and how it gets there:
- [[Data Warehouse]]
- [[Data Lake]]
- [[Data Silo]]
- [[Extract-Transform-Load]]

Who runs the machines:
- [[Cloud Computing]]
- [[Self-Hosting]]
- [[Cloud-Native Architecture]]
- [[Object Storage]]
- [[Separation of Storage and Compute]]
- [[Serverless]]

One machine or many:
- [[Distributed System]]
- [[Node]]
- [[Microservices]]
- [[High-Performance Computing]]

Whose job all of this is:
- [[DevOps]]
- [[Data Engineering]]

Amended, not produced, by this chapter:
- [[Online Learning]], which gained the sentence separating its sense of "online", a model fitted one instance at a time, from the sense the workload names carry, a request answered while the caller waits
- [[Pipeline]], whose existing warning about the data-engineering sense of the word now points at [[Extract-Transform-Load]] for the thing it is warning about
- [[Feature Engineering]], which gained the substrate question: a conformed relational schema reached through SQL is an awkward place to do this work, and stops being an option at all once the input is a photograph

Hubs and indexes revised to match what the chapter delivered, rather than notes it produced:
- [[Data Systems]], the new domain hub
- [[Systems Architecture]], the second domain hub the chapter's material produced, holding what is true of any software system regardless of what it stores
- [[Machine Learning]], the domain hub, whose cross-domain paragraph now names the other domains sitting outside it
- [[MLOps]], whose scope paragraph now carries the boundary against [[DevOps]] and the vocabulary for where anything at all runs
- [[Home]] and the README, which count the sources and the domains

## Open questions

- The four advantages claimed for designing a system for the cloud from the start, better performance on the same hardware, faster recovery, quicker scaling and support for larger datasets, all rest on papers written by the engineers of the systems being measured: Aurora at SIGMOD 2017 with every author at Amazon, Spanner at OSDI 2012 with every author at Google, both in industry tracks, neither with a released configuration. The one independent controlled study points the other way on the first claim, Pang and Wang at Purdue (SIGMOD 2024) measuring storage disaggregation alone at 16.4x on reads and 17.9x on writes on fixed hardware, and nothing published measures scaling latency for these systems at all. [[DDIA]] chapter 2, Defining Nonfunctional Requirements, is the first place that could take this up, because it is where a claim about performance would have to be stated in a form that could be checked.
- Whether a [[Data Warehouse]] and a [[Data Lake]] are still two things. The lakehouse claims to collapse the split, in Armbrust, Ghodsi, Xin and Zaharia at CIDR 2021, which is a paper from the vendor that builds Delta Lake and so stands in the same relation to its subject as the cloud-native papers above. Chapter 1 does not raise it, and [[DDIA]]'s own treatment appears to be deferred to chapter 11, so this is not a question the chapter declined to answer but one it routed elsewhere.
- [[DevOps]] carries a practice, a definition of a job and an economic observation in one note, and it holds together, but the seam is visible. The natural cut is a note on the cost discipline that the shift from capacity planning to financial planning turned into, which now has a name and an organization behind it. The trigger named while writing the note was that the deployment notes would have to exist first, and they now do, so this is live rather than deferred.
- Two domains carry an `Operations/` folder and each holds a single note, [[Data Engineering]] in one and [[DevOps]] in the other, which makes them the thinnest folders either domain has. Chapter 2's operability material is what decides whether either grows into a real area or whether one note is all a folder of that name will ever hold.
- [[Node]] holds a bare and heavily overloaded title. A decision tree node arrives with [[HOML]] chapter 6, and one of the two will have to be qualified. The same collision is already scheduled for another word: [[Model]] is the fitted hypothesis, and [[DDIA]] chapter 3 brings data models.
- Several claims could not be checked against the book itself, because the chapter text is paywalled and only the reference list is public. The sharpest is the statement in [[Hybrid Transactional-Analytical Processing]] that such a system does not replace a data warehouse, which could not be confirmed against Gartner, who coined the term in 2014, or against any vendor running one. It is recorded as the book's claim and not as a checked fact.
- The building blocks the chapter names and does not define, database, cache, search index, batch processing and stream processing, are its largest deliberate deferral. Only the cache has any treatment anywhere in the vault, inside [[Derived Data]], and it is there as the worked example of rebuildability rather than as an account of caching.
- [[Cloud-Native Architecture]] carries a twelve-product roster, of which four names were already wrong or imprecise at the moment they were written down. That note is now doing three jobs at once, defining a stance, auditing the evidence for its claimed advantages and cataloguing products, and the third job is the one with a shelf life. Whether a product roster belongs inside a concept note at all is decidable the next time a chapter arrives carrying one of its own, which on [[DDIA]]'s chapter list is chapter 4, Storage and Retrieval.
- Nothing in the 24 notes fixes a number for "online". [[Online Transaction Processing]] and [[Online Analytical Processing]] both state outright that the term commits to interactivity and to no particular latency, and the product analytics systems tighten it to sub-second by choice. Chapter 2 is where response time, percentiles and targets should arrive, and it is the place to check whether "interactive" ever acquires a number or stays a description of who is waiting.

Settled here, having been asked earlier:

- None outright. All four [[HOML]] chapter notes were checked, and their open questions are about fitting, calibration and optimizers, none of which this chapter touches at any point.
- One is partly settled and is recorded as partly settled. [[MLOps]] asked, as its largest gap, where a trained model actually runs once it leaves a notebook. The vocabulary for where anything at all runs now exists, in [[Cloud Computing]] and [[Self-Hosting]] for who owns the machines, [[Serverless]] for renting the execution rather than the machine, and [[Cloud-Native Architecture]] for what a service has to look like to suit either. The model-specific half, what serving a fitted model asks that serving an ordinary backend service does not, was still missing then and is now partly supplied: [[DMLS Ch01 Overview of Machine Learning Systems]] brought [[Production Machine Learning]], which states four of those asks, several parties each wanting something different instead of one agreed score, a single prediction returned to a caller blocked on it rather than a fit that finishes soon, data that keeps shifting while the fitted artifact stays as it was, and fairness and interpretability becoming things somebody has to account for. What remains is the deployment mechanics, now owed by DMLS chapter 7 as well as [[HOML]] chapter 19.
