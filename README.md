# AI Recruitment Platform

System analysis for the Applicant Tracking System (ATS) covering CV intake, parsing, indexing, search, and ranking. This README covers the First Term scope only.

## The Problem

Recruiters receive a large number of CVs for every job opening. Reviewing them by hand is slow, inconsistent, and tiring.

- CVs come in different formats and layouts, so they are hard to compare.
- Simple keyword matching misses qualified candidates who phrase their experience differently.
- Under time pressure, strong candidates can be overlooked or ranked unfairly.

## The Solution

An Applicant Tracking System (ATS) that automates the first screening step.

- Candidates or recruiters upload CVs, one at a time or in batches.
- The system validates the file, stores it, and extracts structured data from it.
- The structured data is indexed in a vector database to support hybrid search (semantic meaning plus keywords).
- The recruiter enters a Job Description and filters. An LLM rewrites the JD into structured JSON, then the system searches the candidates and scores the matches.
- The recruiter gets a ranked shortlist with match scores and links to the original CV files.

## System Diagrams

### Context Diagram

<p align="center">
  <img src="assets/ATS%20-%20Context%20Diagram.svg" alt="Context Diagram" width="80%">
</p>

- Shows the ATS as one system and the actors around it.
- Candidate uploads a CV.
- Recruiter enters the Job Description and search filters, and uploads CVs (single or batch).
- The system returns a ranked shortlist with match scores to the recruiter.

### Use Case Diagram

<p align="center">
  <img src="assets/ATS%20-%20Use%20Case%20Diagram.svg" alt="Use Case Diagram" width="80%">
</p>

- Actors: Candidate and Recruiter.
- Candidate: Submit Application (upload a single CV).
- Recruiter: Upload CVs (single or batch), Input Job Description (JD), and Search on the top candidates.
- Search includes Rewrite JD via LLM (JSON extraction) and Calculate Match Score (hybrid search and rerank).
- Configure Filtration Mode extends the search. Strict mode removes candidates who do not match the filters. Soft mode keeps them but lowers their score.
- Uploading CVs includes Parse and Extract CV Data.

### DFD Level 1

<p align="center">
  <img src="assets/ATS%20-%20Data%20Flow%20Diagram%20-%20Level%201.svg" alt="DFD Level 1" width="80%">
</p>

- 1.0 Upload and Validate CV: validates the submitted file and saves the original in CV File Storage (Blob Storage).
- 2.0 Parse and Extract Candidate Data: extracts the content and saves structured data in Extracted JSON Storage (NoSQL).
- 3.0 Index Candidate Profile: generates embeddings and stores them with metadata in the Vector Database.
- 4.0 Search and Rank Candidates: takes the recruiter's Job Description, builds the search query, searches the Vector Database, and ranks the matches.
- 5.0 View Match Scores and Candidate Profiles: shows the recruiter the ranked shortlist, match scores, structured profiles, and CV PDF links.

### Activity Diagram

<p align="center">
  <img src="assets/ATS%20-%20Activity%20Diagram.svg" alt="Activity Diagram" width="80%">
</p>

**Upload CV flow**

- Validates file format and size. Invalid files get a Bad Request response.
- Valid files are saved to Blob Storage and the CV status becomes "Uploaded".

**Process CV flow (runs in the background)**

- Extracts the content. If extraction fails, the status becomes "Unvalid File".
- Converts the content to JSON and validates it (is it a CV, is the length acceptable).
- Stores the JSON in NoSQL and sets the status to "Parsed".
- Generates dense and sparse vectors, stores them in the Vector Database, and sets the status to "Indexed".
- Sets the final status to "Ready for search".
- Any failed step is retried, or sent to a Dead Letter Queue (DLQ) if it keeps failing.

**Search flow**

- The recruiter enters the Job Description and metadata filters.
- An LLM rewrites the Job Description.
- Dense and sparse vectors are generated for the query.
- A hybrid search runs on the Vector Database.
- The ranking logic is applied and the top_k candidates are returned.

#### CV Status Lifecycle

| Status | Meaning |
| --- | --- |
| Uploaded | The file passed format and size checks and is stored in Blob Storage. |
| Parsed | The content was extracted, validated as a CV, and saved as JSON in NoSQL. |
| Indexed | Dense and sparse vectors were generated and stored in the Vector Database. |
| Ready for search | Indexing is complete and the CV is available for recruiter searches. |
| Unvalid File | File extraction or content validation failed. |

## First Term Scope

- CV upload (single and batch) with validation
- CV parsing into structured JSON
- Dense and sparse indexing in a vector database
- JD rewriting with an LLM
- Hybrid search with strict or soft filtering
- Ranked shortlist with match scores