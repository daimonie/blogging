```https://xomnia.com/post/navigating-the-european-cloud-landscape-open-telekom-cloud-stackit-and-ovhcloud-in-focus/?utm_content=405874903&utm_medium=social&utm_source=linkedin&hss_channel=lcp-2919540

# It is Time for a Dutch Public Cloud Service
## Introduction 
Cloud Infrastructure has become the foundational layer of modern society. Not only do public services, government institutions, banks and public transport rely on cloud platforms, but generative AI is increasingly integrated into daily business workflows. Most of this runs on commercial American platforms. These hyperscalers are widely used, high-performing and offer production-ready services for almost any imaginable use case. They serve global markets first — and answer to shareholders, not public interests. There is no (local) accountability, public oversight or alignment or even agreement with Dutch or European public interests.

In this blog post, I argue that alongside these commercial platforms, the Netherlands should establish its own public cloud service. Not as a rejection of capitalism or hyperscalers, but as a commons-based infrastructure layer—comparable to our water, road, or electricity networks. A system that guarantees long-term reliability, accessibility, transparency, and democratic governance.

There are two key points I want to make.
- First, that a public version of the cloud is urgently needed—one that prioritizes stability, accessibility, affordability, and public interest over short-term commercial growth.
- Second, that this is technically feasible today, thanks to the maturity of the open-source cloud ecosystem. In the sections that follow, we'll walk through the core needs of a typical cloud platform and show which open-source tools can be used to meet them—and how they fit together.


## What is the cloud?
Most people have now heard of the cloud. It's an ambiguous term - it can be used to refer to working on things like online collaborative documents, but also to online 'cloud' storage, etc. In IT, the term generally refers to hosted services. 

Cloud services are typically divided into private clouds (used by a single organization) and so-called ‘public clouds'—the major commercial platforms like GCP, AWS, and Azure. The term ‘public' here is misleading: it means these services are available to anyone who can pay, not that they are owned or governed by the public. They do offer an enormous range of services needed to run software at scale on the internet. 

Let's use an online store as an example to understand what happens behind the scenes in the cloud. When you visit the store's website, your browser sends a request to a server somewhere—often in a data center operated by one of the major cloud providers. That cloud platform automatically checks if there are enough resources available to handle the incoming traffic. If not, it spins up additional compute instances to meet the demand—this is known as autoscaling.

Once you're connected, your interaction with the store triggers a chain of services. The site may look up whether you're a returning customer, personalize your product recommendations, and fetch real-time pricing and stock information. All of this involves querying databases, running business logic, and rendering pages—possibly across dozens of internal services. Meanwhile, background processes may be running analytics: surfacing trending products, detecting fraud, or predicting supply chain needs. For a large retailer, this could involve hundreds or even thousands of compute nodes working in parallel—each task scheduled, monitored, and scaled dynamically by the cloud infrastructure.

With that example in mind, it becomes clear that many layers of technology are involved in the delivery of these services. Throughout this post, I'll refer to American Clouds - AWS, GCP and Azure - instead of public clouds. I don't do this to critique their origin, but to clearly distinguish them from what I propose: A public cloud in the literal sense of a publicly owned and governed digital infrastructure in addition to commercial platforms.

## What is a public good, a common good, a public service?
To ground the discussion, let's start with some loose economic definitions:
- A public good is something that is non-excludable and non-rivalrous, typically provided by the government and paid for through taxation. 
- A common good, by contrast, is rivalrous - such as fishing in a pond. There's a limited supply and a single person can deplete the resource.

Many public goods - such as railways, electricity grids, and telecom infrastructure - have been partially or fully privatized over the past few decades, with the assumption that market competition would lead to better or cheaper service. In practice, these services are treated as essential and universal. Internet access, for example, is now considered a basic need even though it is delivered by private companies.

I argue that cloud infrastructure - particularly as used in public services, education, and AI - has reached that threshold. It is no longer a convenience, but is critical in the delivery of everything from healthcare to transportation, banking, logistics and government services.

Nearly every citizen interacts with cloud infrastructure, often unconsciously. Mobile apps, online banking, travel cards, healthcare portals and even government websites all run on cloud platforms. This makes the cloud functionally non-excludable - everyone should have access. It's difficult to imagine a future where you're locked out of digital society because you don't have access to a hyperscaler.

But for infrastructure this important, we normally expect democratic safeguards. Today's cloud is fully operated by commercial actors with global reach and zero obligation to the citizens. The risk is not just access restriction or censorship, but subtle forms of platform power - like slower performance for competitors, preferential treatment for corporate partners or quiet deprecation of critical services.

We wouldn't accept this from our road network or our national archives. We shouldn't accept it for the digital infrastructure that underpins them all.

## Why should it be a public good?
If we're pragmatic, the private cloud market currently works. But it doesn't work in the interest of citizens, public agencies, or long-term resilience. 

The Netherlands — and the EU more broadly — has no control over the commercial conditions of the major hyperscalers. If, tomorrow, the U.S. were to introduce a tariff or export restriction that doubled the cost of cloud services to European governments, we would have no recourse. This isn't alarmist- it's what dependency looks like.

We are now in a phase of digital infrastructure where control over data, AI models, and analytic workflows is no longer just a technical concern, but a matter of democratic oversight. These systems shape policy, automate public service delivery, and influence how decisions are made. And let's be realistic: the largest American cloud companies have market valuations that exceed the Dutch government their entire annual budget. We can't compel compliance through regulation alone.

This concern is especially pressing with the rise of generative AI. The Dutch Ministry of the Interior (BZK) recently released a national position that encourages experimentation with genAI in the  public sector - but only with strong guarantees around public values, privacy and responsibility. These guarantees mostly rely on legal contracts with commercial providers. A sovereign cloud platform would provide the technical foundation to enforce those guarantees, not just write them down.

Public infrastructure also matters for innovation. Startups and SMEs often can't afford enterprise-grade cloud services, particularly at scale. A Dutch public cloud could offer discounted or subsidized access in exchange for equity or royalties—similar to a startup loan model. If a startup succeeds, part of the upside flows back into the cloud platform, enabling further investment in infrastructure and innovation.

A comparable model already exists in Dutch academia: when a university researcher patents something, the rights are split three ways—between the inventor, the university, and the university fund. One portion rewards the creator, another covers legal and operational costs, and the final share supports future innovation. A public cloud could adopt a similar framework, reinvesting public-private success into shared digital infrastructure.

## Is it technically feasible?
Absolutely. The tools that power cloud infrastructure today are largely open-source or derived from open-source. American clouds lean heavily on community-developed projects like Kubernetes, Apache Spark and Airflow. The tech underneath is well-known, well-tested and used everywhere from startup to massive-scale corporations.

 Companies like Google, Amazon and Microsoft all use or contribute to open-source projects like [Kubernetes](https://kubernetes.io/docs/concepts/overview/), [Apache Spark](https://spark.apache.org/docs/latest/), [Airflow](https://airflow.apache.org/)and others. What they offer on top is integration, convenience and support - but the underlying technology is available to everyone. They either repackage public innovation into proprietary products, or benefit from free testing at global scale.

By combining these building blocks, we can create a public cloud platform tailored to Dutch needs, with transparency, privacy and interoperability at its core.

## Core components of the Dutch Public Cloud:
Let's break that feasibility claim down into concrete components to run a sovereign cloud platform. These are real tools that modern clouds already use. What is missing is an integration layer; a well-designed interface and developer experience, but this is the skeleton underneath.

Warning: Technical content ahead. If you just want the arguments, feel free to skip to the next section.

Compute and Orchestration
- [Kubernetes](https://kubernetes.io/) for orchestrating workloads across machines
- [Knative](https://knative.dev/docs/) for request-based autoscaling and scale-to-zero serverless behavior
- [OpenFaaS](https://www.openfaas.com/) for function-style execution (like Cloud Functions / Lambda)
- [Kubernetes Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/) and CronJobs for scheduled batch workloads
- [Apache Airflow](https://airflow.apache.org/) for orchestration of various services/resources

API Management and Access:
- [NGINX Ingress](https://github.com/kubernetes/ingress-nginx?tab=readme-ov-file) or [Kong Gateway](https://docs.konghq.com/gateway/latest/) for traffic control, routing and security
- [Keycloak](https://www.keycloak.org/) for identity and access management (IAM)
- [Cert-manager](https://cert-manager.io/) for automatic TLS encryption
- [Istio](https://istio.io/) or [Cillium](https://cilium.io/) for internal service mesh and zero-trust security

Data and Analytics:
- DuckDB and Polars for local, high-speed analytical processing
- Apache Kafka for event-based streaming
- Apache Spark, Flink, and Druid for scalable batch/stream processing
- Trino as a SQL layer over distributed datasets
- MinIO for object-storage with horizontal scaling and encryption
- Apache Superset for data visualization, dashboarding and exploration

AI and LLM Hosting:
- Ollama for running open models locally
- OpenWebUI for managing multi-user chat interfaces with LLMs

Together, this ecosystem provides almost everything needed to run compute, storage, APIs, AI and data pipelines at scale, all under local control. The hard part isn't finding the software, but coordinating it, hosting it reliably, and making it pleasant to use.

Of course, some parts of the modern cloud stack are still hard to replicate with open source alone.
## What about hyperscalers?
We can still leverage some hyperscaler technologies without compromising sovereignty. The trick is to make use of their ecosystem without really exposing our data.

Let's not reinvent every wheel. There's some powerful technology running in the American clouds and some are hard to replace. In particular, none of the options listed above really provide something like BigQuery or Snowflake, and to a lesser degree, Redshift.

One powerful and elegant option is to adopt a model like the company STACKIT from Germany. They reportedly use the services and infrastructure of hyperscalers under the hood, but apply full encryption and sovereign key management to ensure that the third party can't access consumer data, even at infrastructure level.

The approach is especially promising for proprietary data platforms like Snowflake or Bigquery, where no direct open-source alternatives exist that offer the same experience. By layering tools over a hyperscaler backend, we can offer a privacy-first, high performance alternative without compromising sovereignty.

Of course, many institutions already use hybrid approaches, where they process the data without any privacy or identifying information and leverage the power of hyperscalers. It's worth noting that research into anonymization of data has shown that it very often can be reconstructed.

## How would we run this public institution?
American clouds are massive, and that's why they're cost-effective. Scale is a powerful tool for driving down cost, and a public cloud could benefit from the same economies of scale. Yes, there would be upfront investment—setting up data centers, hiring developers to build the integration layer, and deploying the core components—but this isn't uncharted territory.

That investment would be worth it to regain control over our data and infrastructure. As outlined earlier, our reliance on American clouds poses growing risks: pricing changes, degraded performance, contractual uncertainty, and the potential for abrupt policy shifts. This isn't about building a second-rate clone of AWS—it's about reducing structural vulnerability and restoring sovereignty over digital infrastructure. That's a far stronger case than simply increasing regulatory oversight.

I imagine the public cloud as a fully public platform that private actors can still use, much like our electricity grid or national rail network. In that setup, we'd unlock multiple funding streams: existing cloud budgets from government agencies, payments from companies seeking privacy, cost control, or reliability—and yes, probably some ongoing public support. But if hyperscalers can run these services at cost plus profit, I'm confident we can run ours at cost—without needing extravagant subsidies.

## Final Thoughts

Cloud infrastructure is no longer optional. If we believe that public values, transparency, privacy, and national resilience matter, then a Dutch public cloud is a strategic necessity.

We have the talent, the tools, and the momentum. We just need to build it - and own it.```