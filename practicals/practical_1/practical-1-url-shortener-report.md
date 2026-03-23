# Critical Analysis Report
## Chapter 8: Design a URL Shortener
### System Design Interview – Academic Review

---

**Name**  Jigden Shakya 

**Student Number**  02240343

**Module Code**: SDA202

**Module Name**: System Design & Solution Architecture  

---

## Table of Contents

1. Introduction

2. Problem Statement Analysis

3. Analysis of Author's Solutions

4. Personal Review

5. Summary and Critical Analysis

6. References

---

## 1. Introduction

This report is a critical analysis of Chapter 8 from the system design interview book. The chapter covers the design of a URL shortening service similar to TinyURL. The goal of this report is to break down the problem, review the solutions the author proposes, and reflect on what I understood, what confused me, and what I want to look into further.

---



## 2. Problem Statement Analysis

The problem seems simple at first. Take a long URL, return a short one. But the chapter makes it clear early on that the scale of the system is what makes this a real engineering problem.

The author uses a mock interview format to gather requirements before any design work begins. I think this is one of the more important lessons in the chapter because it shows that defining the problem properly matters as much as solving it.

### 2.1 Functional Requirements

- Given a long URL, the system returns a shorter URL
- When the short URL is visited, the user gets redirected to the original URL
- Short URLs consist of alphanumeric characters only (0-9, a-z, A-Z)
- URLs cannot be updated or deleted once created

### 2.2 Non-Functional Requirements

- High availability
- Low latency redirection
- Scalability to handle large traffic volumes

### 2.3 Scale Estimation

The author works through back-of-the-envelope calculations to understand what the system actually needs to handle:

| Metric | Value |
|---|---|
| URLs generated per day | 100 million |
| Write operations per second | ~1,160 |
| Read operations per second | ~11,600 (10:1 ratio) |
| Records over 10 years | 365 billion |
| Storage required | ~365 TB |

These numbers are not just background information. They directly influence design decisions made later in the chapter, particularly around the hash value length and the need for caching.

---

## 3. Analysis of Author's Solutions

### 3.1 API Design

The author proposes two REST API endpoints:

- **POST** `api/v1/data/shorten` — accepts a long URL, returns a short URL
- **GET** `api/v1/shortUrl` — accepts a short URL, returns the original URL with HTTP redirection

This is a minimal and clean design that covers both core use cases without overcomplicating the interface.

### 3.2 URL Redirection: 301 vs 302

![server-client communication](image.png)

The author raises an interesting point about HTTP redirect codes. This is not just a technical detail, it is a product decision.

**301 Permanent Redirect**
- Browser caches the response
- Subsequent requests go directly to the destination, bypassing the shortener
- Reduces server load
- Cannot track repeated clicks on the same link

**302 Temporary Redirect**
- Every click passes through the shortener server
- Enables click tracking and analytics
- Higher server load compared to 301

The choice between them depends on whether the priority is performance or data collection.


### 3.3 URL Shortening
 
The short URL takes the form `www.tinyurl.com/{hashValue}`. To support this, the system needs a hash function    *fx*   that maps a long URL to a hashValue. The function must satisfy two requirements: each longURL hashes to exactly one hashValue, and each hashValue can be mapped back to its original longURL. The detailed design of this function is covered in the deep dive section.
 
### 3.4 Design Deep Dive
 
The high-level design covers the broad strokes of URL shortening and redirecting. This section goes further, examining the data model, hash function, and the shortening and redirecting flows in detail.

### 3.5 Data Model

Rather than using an in-memory hash table, the author recommends a relational database. The reasoning is sound: memory is expensive, not durable, and not practical at the scale of 365 billion records.

The proposed table structure:

| Column | Description |
|---|---|
| id | Auto-incremented primary key |
| shortURL | The generated short link |
| longURL | The original long URL |

### 3.6 Hash Function Design

This is the core of the chapter. The author explores two approaches to generating short URLs.

**Approach 1: Hash + Collision Resolution**

![hash-function and value](image-1.png)

- Apply a hash function (CRC32, MD5, SHA-1) to the long URL
- Take the first 7 characters of the output
- If a collision occurs (same 7 characters for different URLs), append a predefined string and rehash
- Use a Bloom filter to check for collisions efficiently without querying the database each time

![hash and collision resolution flow](image-3.png)

The character set has 62 possible values (0-9, a-z, A-Z). The minimum hash length needed is calculated as the smallest n where 62^n covers 365 billion records. At n=7, 62^7 gives approximately 3.5 trillion combinations which is sufficient.

![hash combinations](image-6.png)

**Approach 2: Base 62 Conversion**

- Each new URL entry receives a unique auto-incremented ID from the database
- That ID is converted from base 10 to base 62
- Example from the chapter: ID 2009215674938 converts to `zn9edcu`
- Because the ID is always unique, the resulting short URL is always unique with no collision possible


![base 62 flow](image-4.png)

**Comparison of Both Approaches:**

| Factor | Hash + Collision Resolution | Base 62 Conversion |
|---|---|---|
| URL length | Fixed | Grows with ID size |
| Collision risk | Possible | None |
| ID generator needed | No | Yes |
| Security concern | Low | Sequential IDs can be guessed |

### 3.5 URL Redirecting Flow

![URL redirecting flow](image-5.png)

The author describes the redirecting architecture using:

- A load balancer to distribute incoming requests
- Web servers to handle logic
- A cache layer checked before the database
- A database as the final source of truth

Since reads are 10 times more frequent than writes, the cache layer is critical for performance. Most redirect requests should be served from cache without touching the database at all.

---

## 4. Personal Review

### 4.1 What I Understood

The part that stood out most to me was how the estimation exercise at the beginning was not just an introduction, it was the foundation for every decision that followed. The reason the hash value is 7 characters long is because 6 characters only supports 56 billion combinations which falls short of the 365 billion record requirement. That direct link between a business requirement and a technical constraint made the whole design feel logical rather than arbitrary.

The 301 vs 302 discussion was also something I had not considered before. It reframes a small HTTP detail into something with real product implications. Depending on whether the business needs analytics or just wants links to work, the answer changes completely.

The base 62 conversion example with actual numbers made the concept easy to follow. Once I understood it as just a different number system using 62 symbols instead of 10, it made complete sense.

### 4.2 Confusions and Unclear Areas

A few things in the chapter were not fully addressed and left open questions for me:

- **Distributed ID Generation:** The base 62 approach depends entirely on unique IDs but the chapter just references Chapter 7 for this. If multiple servers are generating IDs simultaneously, how is uniqueness guaranteed across all of them? This is not a simple problem and it deserved more explanation here.

- **Bloom Filter Accuracy:** Bloom filters can produce false positives, meaning they sometimes report that something exists when it does not. If this happens during collision checking, the system does unnecessary rehashing. The chapter does not mention how often this could occur or what an acceptable false positive rate looks like for a system at this scale.

- **Cache Configuration:** The cache layer is introduced as important but the chapter does not discuss eviction policy, cache size, or failover behavior. For a system handling 11,600 reads per second, these are not minor details.

- **Concurrent Duplicate Requests:** The flowchart checks whether a long URL already exists before creating a new entry. But if two identical requests arrive at the exact same time, both could pass the check and both could create separate records, resulting in two different short URLs for the same long URL. This race condition is not addressed anywhere in the chapter.

### 4.3 Further Topics to Explore

Reading this chapter raised several areas I want to look into more:

- **Custom Short URLs:** Many services allow users to choose their own slug. This design does not support that. How would custom slugs be integrated without conflicting with auto-generated ones?

- **Link Expiry:** Temporary links are common in real products. While adding an expiry field to the database seems straightforward, cleaning up expired entries at the scale of hundreds of billions of records is a much harder operational problem.

- **Analytics Infrastructure:** If 302 redirects are used to track clicks, where does that data go? Writing to a relational database on every redirect at 11,600 requests per second would not scale. An asynchronous processing pipeline with a message queue would likely be needed. Designing that sub-system properly seems like its own interesting problem.

- **Content Safety:** The chapter briefly mentions rate limiting but does not address what happens if a short URL points to harmful content. Should the service validate or scan destination URLs? Keep a blocklist? This seems like an important consideration for any public-facing URL shortener.

---

## 5. Summary and Critical Analysis

Chapter 8 does a good job of walking through URL shortener design in a structured and logical way. The mock interview format sets up the problem well and the estimation math provides a concrete foundation for the design decisions that follow. The comparison between hash-based generation and base 62 conversion is one of the stronger parts of the chapter because it presents both options honestly without oversimplifying.

Where the chapter falls short is in the depth of certain components. The distributed ID generator, Bloom filter behavior, cache configuration and concurrent request handling are all either skipped or deferred to other parts of the book. For interview preparation purposes this may be acceptable, but as a complete reference for building such a system it leaves noticeable gaps.

The most important takeaway from this chapter is not any specific technical solution but the approach of connecting every design decision back to a concrete requirement. The 7-character hash length exists because of the 365 billion record estimate. The cache exists because reads outnumber writes 10 to 1. This kind of requirement-driven design thinking is something that applies far beyond URL shorteners.

---

## 6. References

- Xu, A. (2020). *System Design Interview – An Insider's Guide*. Chapter 8: Design a URL Shortener.
- REST API Tutorial: https://www.restapitutorial.com/index.html
- Bloom Filter – Wikipedia: https://en.wikipedia.org/wiki/Bloom_filter