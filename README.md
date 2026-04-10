Technical Specification: Flat-Chained TLV Storage (FCT)

Version: 1.0 (Draft for Defensive Publication)
Status: Design Phase
1. Abstract

This paper proposes Flat-Chained TLV (FCT), a lightweight, non-relational binary storage architecture designed for resource-constrained embedded systems and mobile environments. By leveraging filesystem metadata as a logical index and implementing a flat branching strategy to handle record updates, FCT eliminates the need for complex database engines while maintaining high performance and data integrity for rich-text and multimedia logging.
2. Core Architecture
2.1 Binary Data Unit (TLV)

Each data block follows a strict Type-Length-Value (TLV) format to ensure binary safety and minimize overhead:

Type (1 byte): Defines data category (Text, Image, Metadata, etc.).
Length (4 bytes): 32-bit unsigned integer defining the value size.
Value (N bytes): Raw payload data.

*Note: Temporal information is not mandated at the protocol level. It can be implemented as a specific metadata-type block or managed by the filesystem's modification time to achieve the smallest possible header overhead.*

2.2 Physical Segmentation (The Hard-Limit)

Data is stored in physical files with a predefined Hard-Limit (e.g., 5MB - 10MB).

    Trunk File: The initial record segment, named [RecordID].dat.

    Branch Files: Subsequent segments triggered by data overflow or updates, named [RecordID].[n].dat (where n is an incremental integer).

3. Logic Chain & Update Strategy
3.1 Filename-Based Indexing

FCT delegates the indexing overhead to the host filesystem. The logical reconstruction of a record is achieved by:

    Scanning the directory for [RecordID].*.

    Sorting files via natural numerical order.

    Streaming the binary contents into a contiguous memory buffer for rendering.

3.2 Flat Branching vs. Recursive Nesting

To prevent performance degradation (the "branch-of-branch" problem), FCT enforces a Flat Strategy:

    If a branch (e.g., ID.1.dat) requires an update that exceeds the hard limit, the system merges the branch data in memory and re-slices it into sequential flat segments (ID.1.dat, ID.2.dat).

    Recursive depth is strictly limited to 1, ensuring predictable IOPS and low latency.

4. Implementation Goals

- Deterministic Write IO: Updates only affect the targeted branch file, significantly reducing write amplification on Flash storage.
- Zero-Index Memory Footprint: No external B-Tree or WAL files are required, making it ideal for low-RAM environments.
- High Streaming Efficiency: The flat-chained structure allows for sequential data streaming and easy integration with memory-mapped file (mmap) access for rapid data loading.

5. Defensive Statement

The FCT protocol is designed for data sovereignty and offline-first applications. This specification is publicly disclosed as of April 2026 to establish prior art. The scope of this disclosure includes, but is not limited to:
- The filename-based logic chain mechanism (ID.n.dat).
- The flat branching strategy to prevent recursive nesting.
- The method of localized re-writes of physical segments to handle logical updates.

This disclosure aims to ensure the freedom for any developer to implement, modify, and distribute systems based on these concepts without fear of patent encumbrance from third parties.
