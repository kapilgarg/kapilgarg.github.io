---
title: Cost of workarounds
slug: cost-or-workarounds
date_published: 2020-12-31T06:31:42.000Z
date_updated: 2020-12-31T06:31:42.000Z
tags: softwaredevelopment
---

**W**hen we use a product for something other than it was intended, we have to make some workarounds. Otherwise it wouldn't work. People working on hacking or making the product fit in new scope may get the intellectual satisfaction but ultimately it is going to cost a lot that we don’t realize at that time. Specially if this is done primarily because of cost benefit. It costs in the form of *increased complexity*.

**I**n the context of software development, we write code to patch / update the behavior of product. This causes a lot of additional code, many corner cases left open, design conflicts etc. Developers have to spend a lot of time making sure that the product is supporting the new usage. This becomes the maintenance overhead the moment software is deployed. These additional costs may not be apparent initially. But with time, it goes up. With every product update, we need to validate these workarounds, thus forcing continuous work. While this would not have been required if the product was doing what it was intended for.

*just to get an idea : *AWS Lambda may not be a good choice for service which is expected to provide response with minimum latency (few ms). Lambda needs cold start. So we implement a workaround to keep the lambda warm.

Using S3 as a replacement for relational database since we can execute SQL queries with Athena. Then we end up writing code to handle atomic transaction, high availability , backup-recovery etc. This is just an example but you get the point!!!

Technology decisions made early in the lifecycle have a huge impact over the life of product. We should be very careful when choosing a product or service to do something which is not the primary use case for it. Given the speed at which technology changes, product features are implemented, these workarounds *may *be OK for short term but the product design should not be based on this.

---
