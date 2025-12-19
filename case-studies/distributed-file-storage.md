# Case Study: Distributed File Storage (Advanced)

## Problem Statement

Design a distributed file storage and synchronization system like Dropbox or Google Drive that allows users to store files in the cloud, sync them across multiple devices, share files with others, handle conflicts when the same file is edited offline on different devices, and ensure data durability and availability at massive scale.

## Requirements Analysis

### Functional Requirements

**Core Features:**
1. **File Operations**:
   - Upload files and folders
   - Download files
   - Delete and restore files (trash)
   - Rename and move files
   - Create folders

2. **Synchronization**:
   - Auto-sync files across devices
   - Detect and sync changes in real-time
   - Handle offline changes
   - Conflict resolution for simultaneous edits
   - Selective sync (choose what to sync)

3. **Sharing & Collaboration**:
   - Share files/folders with users
   - Public links with expiration
   - Permissions (view, edit, comment)
   - Real-time collaboration on documents
   - Version history

4. **File Management**:
   - Search files by name, content, metadata
   - Preview files in browser
   - File versioning (keep old versions)
   - Recover deleted files (30-day retention)
   - Storage quota management

5. **Security**:
   - Encryption at rest and in transit
   - Two-factor authentication
   - Device management
   - Audit logs

### Non-Functional Requirements

**Scale Requirements:**
- **Users**: 500 million registered users
  - 100 million daily active users (DAU)
  - 10 million concurrent users during peak
- **Files**: 10 billion files stored
- **Storage**: 10 exabytes (10,000 PB) total
  - Average 20 GB per user
- **Traffic**: 
  - 1 million file uploads per minute
  - 5 million file downloads per minute
  - 100 million file syncs per minute

**Performance Requirements:**
- **Upload Speed**: Limited by user bandwidth
- **Download Speed**: Serve from nearby edge (< 100ms latency to CDN)
- **Sync Latency**: < 5 seconds for small changes
- **Conflict Detection**: < 1 second
- **File Metadata Operations**: < 100ms
- **Availability**: 99.99% uptime

**Data Durability:**
- 99.999999999% (11 nines) durability
- Never lose data even with multiple failures
- Geographic redundancy

**Consistency:**
- Eventual consistency for file metadata
- Strong consistency for file content
- Conflict-free replicated data types (CRDTs) for collaborative editing

## Capacity Estimation

### Storage Calculation

```
**User Files:**
500M users × 20 GB average = 10 EB (10,000 PB)

**With Replication (3x for durability):**
10 EB × 3 = 30 EB

**Version History (assume 20% overhead):**
10 EB × 1.2 = 12 EB
With replication: 36 EB

**Metadata:**
10B files × 2 KB metadata = 20 TB
Negligible compared to file storage

**Total Storage:** 36 EB with versioning and replication
```

### Traffic Estimation

```
**Upload Traffic:**
1M uploads/min = 16,700/sec
Average file size: 10 MB
16,700 × 10 MB = 167 GB/sec peak

**Download Traffic:**
5M downloads/min = 83,300/sec
83,300 × 10 MB = 833 GB/sec peak

**Sync Operations (metadata only):**
100M sync checks/min = 1.67M/sec
Metadata: 1.67M × 2 KB = 3.3 GB/sec

**Total Bandwidth:**
Ingress: 167 GB/sec
Egress: 836 GB/sec (mostly served from CDN)
```

### Database Load

```
**Metadata Operations:**
File operations: 2M/sec
Folder operations: 500K/sec
Sync checks: 1.67M/sec
Total: 4.17M metadata operations/sec

**Storage:**
Metadata DB: 20 TB (manageable with sharding)
User DB: 1 TB
Sharing/Permissions: 5 TB
```

## High-Level Architecture

```
┌───────────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER                                  │
│  [Desktop Sync Clients] [Mobile Apps] [Web Apps] [API Clients]       │
└────────────────────────────────┬──────────────────────────────────────┘
                                 │
┌────────────────────────────────▼──────────────────────────────────────┐
│                    CDN / EDGE STORAGE                                  │
│         [CloudFront / Cloudflare] - 200+ Edge Locations               │
│              - Popular files cached at edge                           │
│              - 90% of downloads served from edge                      │
└────────────────────────────────┬──────────────────────────────────────┘
                                 │
┌────────────────────────────────▼──────────────────────────────────────┐
│                    API GATEWAY / LOAD BALANCER                         │
│         Authentication, Rate limiting, Request routing                │
└──────┬────────────────┬─────────────────┬─────────────────┬──────────┘
       │                │                 │                 │
       ▼                ▼                 ▼                 ▼
┌────────────┐  ┌───────────────┐  ┌────────────┐  ┌──────────────┐
│   FILE     │  │   SYNC        │  │  METADATA  │  │   SHARING    │
│  SERVICE   │  │   SERVICE     │  │  SERVICE   │  │   SERVICE    │
│ - Upload   │  │ - Monitor     │  │ - File ops │  │ - Permissions│
│ - Download │  │   changes     │  │ - Search   │  │ - Links      │
│ - Chunking │  │ - Push updates│  │ - Versions │  │ - Collab     │
└─────┬──────┘  └───────┬───────┘  └──────┬─────┘  └──────┬───────┘
      │                 │                 │                │
      ▼                 ▼                 ▼                ▼
┌───────────────────────────────────────────────────────────────────────┐
│                    SUPPORTING SERVICES                                 │
├──────────────┬──────────────┬──────────────┬──────────────┬──────────┤
│ DEDUPLICATION│ NOTIFICATION │ SEARCH       │ ANALYTICS    │ SECURITY │
│ - Chunking   │ - Push/Email │ - Full-text  │ - Usage      │ - Encrypt│
│ - Hashing    │ - WebSocket  │ - Elastic    │ - ML         │ - Scan   │
└──────┬───────┴──────┬───────┴──────┬───────┴──────┬───────┴────┬─────┘
       │              │              │              │            │
┌──────▼──────────────▼──────────────▼──────────────▼────────────▼──────┐
│                    MESSAGE QUEUE (Kafka)                               │
│  [File Events] [Sync Events] [Share Events] [Analytics Events]       │
└────────────────────────────────┬───────────────────────────────────────┘
                                 │
┌────────────────────────────────▼───────────────────────────────────────┐
│                          DATA LAYER                                     │
├──────────────┬──────────────┬──────────────┬──────────────┬───────────┤
│ METADATA DB  │ BLOCK STORAGE│ USER DB      │ CACHE        │ SEARCH    │
│ (Cassandra)  │ (S3/Block)   │ (PostgreSQL) │ (Redis)      │(Elastic)  │
│ - File meta  │ - File chunks│ - Users      │ - Sessions   │- File idx │
│ - Versions   │ - Blocks     │ - Devices    │ - Hot files  │- Content  │
│ - Tree       │ - Dedup map  │ - Sharing    │              │           │
└──────────────┴──────────────┴──────────────┴──────────────┴───────────┘

┌───────────────────────────────────────────────────────────────────────┐
│              OBJECT STORAGE (S3) - 36 EB                               │
│  Multi-region, Multi-AZ, Erasure coded for durability                 │
└───────────────────────────────────────────────────────────────────────┘
```

## File Chunking & Deduplication

### Chunking Strategy

**Fixed-Size Chunking:**
```python
def chunk_file_fixed(file_path: str, chunk_size: int = 4194304) -> list:
    """Split file into fixed-size chunks (4 MB default)."""
    chunks = []
    with open(file_path, 'rb') as f:
        chunk_index = 0
        while True:
            chunk_data = f.read(chunk_size)
            if not chunk_data:
                break
            
            # Calculate hash for deduplication
            chunk_hash = hashlib.sha256(chunk_data).hexdigest()
            
            chunks.append({
                'index': chunk_index,
                'size': len(chunk_data),
                'hash': chunk_hash,
                'data': chunk_data
            })
            chunk_index += 1
    
    return chunks
```

**Variable-Size Chunking (CDC - Content-Defined Chunking):**
```python
class ContentDefinedChunking:
    def __init__(self, avg_chunk_size: int = 4194304):
        self.avg_size = avg_chunk_size
        self.min_size = avg_chunk_size // 4
        self.max_size = avg_chunk_size * 4
        self.mask = (1 << 13) - 1  # 8 KB rolling window
    
    def chunk_file(self, file_path: str) -> list:
        """Split file using content-defined boundaries (better dedup)."""
        chunks = []
        with open(file_path, 'rb') as f:
            chunk_data = bytearray()
            chunk_index = 0
            rolling_hash = 0
            
            while True:
                byte = f.read(1)
                if not byte:
                    # Last chunk
                    if chunk_data:
                        chunks.append(self._create_chunk(chunk_data, chunk_index))
                    break
                
                chunk_data.extend(byte)
                rolling_hash = ((rolling_hash << 1) + ord(byte)) & self.mask
                
                # Check if boundary condition met
                if (len(chunk_data) >= self.min_size and 
                    rolling_hash % self.avg_size == 0) or \
                   len(chunk_data) >= self.max_size:
                    # Chunk boundary found
                    chunks.append(self._create_chunk(chunk_data, chunk_index))
                    chunk_data = bytearray()
                    chunk_index += 1
                    rolling_hash = 0
        
        return chunks
    
    def _create_chunk(self, data: bytearray, index: int) -> dict:
        """Create chunk with hash."""
        chunk_hash = hashlib.sha256(data).hexdigest()
        return {
            'index': index,
            'size': len(data),
            'hash': chunk_hash,
            'data': bytes(data)
        }
```

### Deduplication System

```python
class DeduplicationService:
    def __init__(self):
        self.db = Database()
        self.storage = ObjectStorage()
    
    async def upload_file_with_dedup(
        self,
        user_id: str,
        file_path: str,
        chunks: list
    ) -> dict:
        """Upload file with chunk-level deduplication."""
        file_id = str(uuid.uuid4())
        uploaded_chunks = []
        dedup_savings = 0
        
        for chunk in chunks:
            # Check if chunk already exists (by hash)
            existing = await self.db.query(
                "SELECT chunk_id FROM chunks WHERE hash = ?",
                chunk['hash']
            )
            
            if existing:
                # Chunk already exists - just reference it
                chunk_id = existing['chunk_id']
                dedup_savings += chunk['size']
            else:
                # New chunk - upload to storage
                chunk_id = str(uuid.uuid4())
                await self.storage.put(
                    f"chunks/{chunk_id}",
                    chunk['data']
                )
                
                # Store chunk metadata
                await self.db.execute(
                    """
                    INSERT INTO chunks (chunk_id, hash, size, ref_count)
                    VALUES (?, ?, ?, 1)
                    """,
                    chunk_id, chunk['hash'], chunk['size']
                )
            
            # Record chunk for this file
            uploaded_chunks.append({
                'chunk_id': chunk_id,
                'index': chunk['index'],
                'size': chunk['size'],
                'hash': chunk['hash']
            })
            
            # Increment reference count
            await self.db.execute(
                "UPDATE chunks SET ref_count = ref_count + 1 WHERE chunk_id = ?",
                chunk_id
            )
        
        # Create file metadata
        await self.create_file_metadata(
            file_id,
            user_id,
            file_path,
            uploaded_chunks
        )
        
        return {
            'file_id': file_id,
            'total_size': sum(c['size'] for c in chunks),
            'actual_uploaded': sum(c['size'] for c in chunks) - dedup_savings,
            'dedup_savings_bytes': dedup_savings,
            'dedup_ratio': dedup_savings / sum(c['size'] for c in chunks) if chunks else 0
        }
    
    async def delete_file_with_dedup(self, file_id: str):
        """Delete file and cleanup unreferenced chunks."""
        # Get file's chunks
        chunks = await self.get_file_chunks(file_id)
        
        # Delete file metadata
        await self.db.execute("DELETE FROM files WHERE file_id = ?", file_id)
        await self.db.execute("DELETE FROM file_chunks WHERE file_id = ?", file_id)
        
        # Decrement chunk reference counts
        for chunk in chunks:
            result = await self.db.execute(
                """
                UPDATE chunks 
                SET ref_count = ref_count - 1 
                WHERE chunk_id = ?
                RETURNING ref_count
                """,
                chunk['chunk_id']
            )
            
            # If no more references, delete chunk from storage
            if result['ref_count'] == 0:
                await self.storage.delete(f"chunks/{chunk['chunk_id']}")
                await self.db.execute(
                    "DELETE FROM chunks WHERE chunk_id = ?",
                    chunk['chunk_id']
                )
```

### Delta Sync (Upload Only Changes)

```python
class DeltaSyncService:
    async def sync_file_changes(
        self,
        file_id: str,
        old_chunks: list,
        new_chunks: list
    ) -> dict:
        """Upload only changed chunks (rsync-style)."""
        # Create hash maps for quick lookup
        old_hashes = {c['hash']: c for c in old_chunks}
        new_hashes = {c['hash']: c for c in new_chunks}
        
        # Find chunks to upload (present in new, not in old)
        chunks_to_upload = [
            chunk for chunk in new_chunks
            if chunk['hash'] not in old_hashes
        ]
        
        # Find chunks to delete (present in old, not in new)
        chunks_to_delete = [
            chunk for chunk in old_chunks
            if chunk['hash'] not in new_hashes
        ]
        
        # Upload new chunks
        for chunk in chunks_to_upload:
            await self.upload_chunk(chunk)
        
        # Update file metadata with new chunk list
        await self.update_file_chunks(file_id, new_chunks)
        
        # Cleanup old chunks
        for chunk in chunks_to_delete:
            await self.decrement_chunk_refcount(chunk['chunk_id'])
        
        return {
            'chunks_uploaded': len(chunks_to_upload),
            'chunks_reused': len(new_chunks) - len(chunks_to_upload),
            'bandwidth_saved': sum(c['size'] for c in new_chunks) - 
                             sum(c['size'] for c in chunks_to_upload)
        }
```

## Conflict Resolution

### Conflict Detection

```python
class ConflictDetector:
    async def detect_conflict(
        self,
        file_id: str,
        device_id: str,
        local_version: int,
        local_modified_at: float
    ) -> dict:
        """Detect if file has conflicting changes."""
        # Get current server version
        server_file = await self.db.query(
            """
            SELECT version, modified_at, modified_by_device
            FROM files
            WHERE file_id = ?
            """,
            file_id
        )
        
        if not server_file:
            return {'conflict': False, 'reason': 'new_file'}
        
        # Check for conflict
        if local_version < server_file['version']:
            # Server has newer version
            if local_modified_at > server_file['modified_at']:
                # Both client and server modified after last sync - CONFLICT!
                return {
                    'conflict': True,
                    'reason': 'concurrent_modification',
                    'server_version': server_file['version'],
                    'local_version': local_version,
                    'server_modified_at': server_file['modified_at'],
                    'local_modified_at': local_modified_at
                }
            else:
                # Server is newer, client is stale - no conflict, just update
                return {
                    'conflict': False,
                    'reason': 'stale_local',
                    'action': 'pull_from_server'
                }
        else:
            # Client is up to date or newer
            return {'conflict': False, 'reason': 'client_newer'}
```

### Conflict Resolution Strategies

**Strategy 1: Last-Write-Wins (LWW)**
```python
async def resolve_lww(file_id: str, local_mod: float, server_mod: float):
    """Simple but potentially loses data."""
    if local_mod > server_mod:
        # Keep local version
        await upload_local_version(file_id)
    else:
        # Keep server version
        await download_server_version(file_id)
```

**Strategy 2: Keep Both (Conflict Files)**
```python
class ConflictResolver:
    async def resolve_keep_both(
        self,
        file_id: str,
        file_name: str,
        device_id: str,
        local_content: bytes
    ):
        """Create conflict copy - what Dropbox does."""
        # Get server version
        server_content = await self.download_file(file_id)
        
        # Keep server version as-is
        # Create conflicted copy with local changes
        conflict_name = self.generate_conflict_name(
            file_name,
            device_id
        )
        # "document.txt" -> "document (Device's conflicted copy 2024-01-15).txt"
        
        conflict_file_id = await self.create_file(
            conflict_name,
            local_content
        )
        
        # Notify user
        await self.notify_user(
            f"Conflict detected. Created {conflict_name}"
        )
    
    def generate_conflict_name(self, filename: str, device: str) -> str:
        """Generate conflict file name."""
        name, ext = os.path.splitext(filename)
        timestamp = datetime.now().strftime('%Y-%m-%d')
        return f"{name} ({device}'s conflicted copy {timestamp}){ext}"
```

**Strategy 3: Three-Way Merge (Advanced)**
```python
class ThreeWayMerge:
    async def merge_text_file(
        self,
        file_id: str,
        base_content: str,
        local_content: str,
        server_content: str
    ) -> dict:
        """Merge text files using three-way merge (like Git)."""
        # Use difflib or similar library
        local_diff = self.compute_diff(base_content, local_content)
        server_diff = self.compute_diff(base_content, server_content)
        
        # Check if changes overlap
        if self.changes_conflict(local_diff, server_diff):
            # Cannot auto-merge - create conflict markers
            merged = self.create_conflict_markers(
                local_content,
                server_content
            )
            return {
                'success': False,
                'merged_content': merged,
                'requires_manual_resolution': True
            }
        else:
            # Changes don't overlap - auto-merge
            merged = self.apply_both_diffs(base_content, local_diff, server_diff)
            return {
                'success': True,
                'merged_content': merged,
                'requires_manual_resolution': False
            }
    
    def create_conflict_markers(self, local: str, server: str) -> str:
        """Create Git-style conflict markers."""
        return f"""<<<<<<< Local version
{local}
=======
{server}
>>>>>>> Server version
"""
```

**Strategy 4: Operational Transform (Real-Time Collaboration)**
```python
class OperationalTransform:
    """For real-time collaborative editing (Google Docs style)."""
    
    def transform_operations(
        self,
        op1: dict,
        op2: dict,
        base_version: int
    ) -> tuple:
        """Transform two concurrent operations to apply sequentially."""
        # Example operations:
        # op1: insert('hello', position=0)
        # op2: insert('world', position=0)
        # Both at same position - need to transform
        
        if op1['type'] == 'insert' and op2['type'] == 'insert':
            if op1['position'] <= op2['position']:
                # op2 happens after op1, adjust position
                op2_transformed = {
                    'type': 'insert',
                    'text': op2['text'],
                    'position': op2['position'] + len(op1['text'])
                }
                return (op1, op2_transformed)
            else:
                # op1 happens after op2, adjust position
                op1_transformed = {
                    'type': 'insert',
                    'text': op1['text'],
                    'position': op1['position'] + len(op2['text'])
                }
                return (op1_transformed, op2)
        
        # Handle delete, replace, etc.
        # ... more transformation logic
```

## Synchronization Protocol

### Sync Algorithm

```python
class SyncService:
    async def sync_folder(
        self,
        user_id: str,
        device_id: str,
        folder_path: str
    ):
        """Sync folder with server."""
        # 1. Get local state (file list with hashes and versions)
        local_state = await self.get_local_state(folder_path)
        
        # 2. Get server state
        server_state = await self.get_server_state(user_id, folder_path)
        
        # 3. Compute differences
        to_upload = []  # Files that exist locally but not on server
        to_download = []  # Files that exist on server but not locally
        to_update_local = []  # Files that are newer on server
        to_update_server = []  # Files that are newer locally
        conflicts = []  # Files modified on both sides
        
        # Compare local and server
        for file_path, local_file in local_state.items():
            server_file = server_state.get(file_path)
            
            if not server_file:
                # New local file
                to_upload.append(local_file)
            elif local_file['hash'] != server_file['hash']:
                # File changed - check which is newer
                if local_file['version'] > server_file['version']:
                    to_update_server.append(local_file)
                elif local_file['version'] < server_file['version']:
                    to_update_local.append(server_file)
                else:
                    # Same version but different content - conflict!
                    conflicts.append((local_file, server_file))
        
        # Check for server files not in local
        for file_path, server_file in server_state.items():
            if file_path not in local_state:
                to_download.append(server_file)
        
        # 4. Execute sync operations
        await self.execute_sync(
            to_upload,
            to_download,
            to_update_local,
            to_update_server,
            conflicts
        )
    
    async def execute_sync(
        self,
        to_upload: list,
        to_download: list,
        to_update_local: list,
        to_update_server: list,
        conflicts: list
    ):
        """Execute sync operations."""
        # Upload new files
        for file in to_upload:
            await self.upload_file(file)
        
        # Download new files
        for file in to_download:
            await self.download_file(file)
        
        # Update local files (server is newer)
        for file in to_update_local:
            await self.download_file(file, overwrite=True)
        
        # Update server files (local is newer)
        for file in to_update_server:
            await self.upload_file(file, overwrite=True)
        
        # Resolve conflicts
        for local_file, server_file in conflicts:
            await self.resolve_conflict(local_file, server_file)
```

### Real-Time Sync with WebSocket

```python
class RealTimeSyncService:
    async def watch_for_changes(self, user_id: str, device_id: str):
        """Push changes to client in real-time."""
        # Establish WebSocket connection
        ws = await self.websocket_manager.connect(device_id)
        
        # Subscribe to user's file change events
        async for event in self.kafka.consume(f"user_{user_id}_changes"):
            # Don't send changes back to the device that made them
            if event['device_id'] == device_id:
                continue
            
            # Push change notification to client
            await ws.send_json({
                'type': 'file_changed',
                'file_id': event['file_id'],
                'action': event['action'],  # created, modified, deleted
                'version': event['version'],
                'modified_by': event['device_id']
            })
            
            # Client can then pull the changed file
```

## API Design

### 1. Upload File

```http
POST /api/v1/files/upload/initiate
Authorization: Bearer <token>
Content-Type: application/json

Request:
{
  "file_name": "document.pdf",
  "file_size": 10485760,
  "parent_folder_id": "folder_123",
  "chunks": [
    {"index": 0, "size": 4194304, "hash": "abc123..."},
    {"index": 1, "size": 4194304, "hash": "def456..."},
    {"index": 2, "size": 2097152, "hash": "ghi789..."}
  ]
}

Response 200 OK:
{
  "file_id": "file_xyz789",
  "upload_id": "upload_abc123",
  "chunks_needed": [0, 2],  // Chunk 1 already exists (dedup)
  "upload_urls": [
    {
      "chunk_index": 0,
      "url": "https://s3.amazonaws.com/upload/chunk0?signature=..."
    },
    {
      "chunk_index": 2,
      "url": "https://s3.amazonaws.com/upload/chunk2?signature=..."
    }
  ]
}
```

### 2. Download File

```http
GET /api/v1/files/{file_id}/download
Authorization: Bearer <token>

Response 200 OK:
Content-Disposition: attachment; filename="document.pdf"
Content-Length: 10485760

{binary file data}

// Or for large files, return chunk URLs
Response 200 OK:
{
  "file_id": "file_xyz789",
  "file_name": "document.pdf",
  "total_size": 10485760,
  "chunks": [
    {
      "index": 0,
      "size": 4194304,
      "url": "https://cdn.example.com/chunks/chunk0"
    },
    {
      "index": 1,
      "size": 4194304,
      "url": "https://cdn.example.com/chunks/chunk1"
    },
    {
      "index": 2,
      "size": 2097152,
      "url": "https://cdn.example.com/chunks/chunk2"
    }
  ]
}
```

### 3. Get Sync Delta

```http
GET /api/v1/sync/delta?cursor=cursor_abc123&limit=1000
Authorization: Bearer <token>

Response 200 OK:
{
  "changes": [
    {
      "file_id": "file_123",
      "path": "/Documents/report.pdf",
      "action": "created",
      "version": 1,
      "modified_at": "2024-01-15T10:30:00Z",
      "size": 1048576,
      "hash": "abc123..."
    },
    {
      "file_id": "file_456",
      "path": "/Photos/image.jpg",
      "action": "modified",
      "version": 3,
      "modified_at": "2024-01-15T10:35:00Z"
    },
    {
      "file_id": "file_789",
      "path": "/temp.txt",
      "action": "deleted",
      "version": 2,
      "modified_at": "2024-01-15T10:40:00Z"
    }
  ],
  "next_cursor": "cursor_def456",
  "has_more": true
}
```

### 4. Share File

```http
POST /api/v1/files/{file_id}/share
Authorization: Bearer <token>
Content-Type: application/json

Request:
{
  "users": ["user_123", "user_456"],
  "permission": "edit",  // view, edit, comment
  "notify": true,
  "message": "Please review this document"
}

Response 200 OK:
{
  "share_id": "share_abc123",
  "file_id": "file_xyz789",
  "shared_with": [
    {"user_id": "user_123", "permission": "edit"},
    {"user_id": "user_456", "permission": "edit"}
  ],
  "created_at": "2024-01-15T10:30:00Z"
}
```

## Database Schema

### Files Metadata (Cassandra)

```sql
CREATE TABLE files (
    file_id UUID PRIMARY KEY,
    user_id UUID,
    parent_folder_id UUID,
    file_name TEXT,
    file_path TEXT,
    size BIGINT,
    hash TEXT,
    version INT,
    created_at TIMESTAMP,
    modified_at TIMESTAMP,
    modified_by_device UUID,
    is_deleted BOOLEAN DEFAULT FALSE,
    deleted_at TIMESTAMP
);

-- Index by user and path
CREATE TABLE files_by_user_path (
    user_id UUID,
    file_path TEXT,
    file_id UUID,
    version INT,
    PRIMARY KEY (user_id, file_path)
);

-- Index for sync (changes since last cursor)
CREATE TABLE file_changes (
    user_id UUID,
    change_id TIMEUUID,
    file_id UUID,
    action TEXT,  // created, modified, deleted
    version INT,
    PRIMARY KEY (user_id, change_id)
) WITH CLUSTERING ORDER BY (change_id DESC);
```

### Chunks Table

```sql
CREATE TABLE chunks (
    chunk_id UUID PRIMARY KEY,
    hash TEXT UNIQUE,
    size INT,
    storage_path TEXT,
    ref_count INT,  // Number of files referencing this chunk
    created_at TIMESTAMP
);

CREATE TABLE file_chunks (
    file_id UUID,
    chunk_index INT,
    chunk_id UUID,
    PRIMARY KEY (file_id, chunk_index)
);
```

### Sharing & Permissions (PostgreSQL)

```sql
CREATE TABLE shares (
    share_id UUID PRIMARY KEY,
    file_id UUID REFERENCES files(file_id),
    owner_id UUID,
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP
);

CREATE TABLE share_permissions (
    share_id UUID REFERENCES shares(share_id),
    user_id UUID,
    permission VARCHAR(20),  -- view, edit, comment
    granted_at TIMESTAMP DEFAULT NOW(),
    
    PRIMARY KEY (share_id, user_id)
);

CREATE TABLE public_links (
    link_id UUID PRIMARY KEY,
    file_id UUID REFERENCES files(file_id),
    token VARCHAR(255) UNIQUE,
    password_hash TEXT,
    expires_at TIMESTAMP,
    download_count INT DEFAULT 0,
    max_downloads INT,
    created_at TIMESTAMP DEFAULT NOW()
);
```

## Trade-Off Discussions

### 1. Chunking: Fixed vs Variable Size

**Fixed-Size Chunking:**
- ✅ Simple implementation
- ✅ Predictable performance
- ❌ Poor deduplication (inserting bytes shifts all chunks)

**Variable-Size (CDC):**
- ✅ Better deduplication (only affected chunks change)
- ✅ Efficient delta sync
- ❌ More complex
- ❌ Variable performance

**Recommendation**: Variable-size chunking (CDC) for better deduplication despite complexity.

### 2. Sync: Poll vs Push

**Polling (Client checks periodically):**
- ✅ Simple, no persistent connections
- ✅ Works with firewalls
- ❌ Delays in updates (poll interval)
- ❌ Wasted requests if no changes

**Push (WebSocket notifications):**
- ✅ Real-time updates
- ✅ Efficient (only when changes occur)
- ❌ Requires persistent connections
- ❌ More complex infrastructure

**Recommendation**: Push for real-time sync, poll as fallback.

### 3. Conflict Resolution: Auto vs Manual

**Automatic (Keep Both):**
- ✅ Never loses data
- ✅ Simple for users
- ❌ Creates clutter (conflict files)

**Manual (User chooses):**
- ✅ User control
- ✅ Cleaner file tree
- ❌ Requires user intervention
- ❌ Blocks sync

**Recommendation**: Auto (keep both) as default, provide tools for manual merge.

### 4. Storage: Block vs Object

**Block Storage (EBS):**
- ✅ Fast random access
- ✅ Good for databases
- ❌ Expensive at scale
- ❌ Single-region

**Object Storage (S3):**
- ✅ Cost-effective at scale
- ✅ Durable (11 nines)
- ✅ Multi-region
- ❌ Higher latency

**Recommendation**: Object storage (S3) with CDN for file chunks, block storage for databases.

## Scaling Strategies

### Phase 1: Startup (< 10K users)

```
[Client Apps] → [Monolith API] → [PostgreSQL] → [S3]
```

- Monolithic application
- Single database
- Simple sync (full file uploads)
- S3 for storage

### Phase 2: Growth (10K - 1M users)

```
[Client Apps] → [API Gateway]
                     ↓
[Microservices: File, Sync, Share]
                     ↓
[PostgreSQL Cluster] + [Redis Cache] + [S3]
```

- Microservices architecture
- Database read replicas
- Redis for metadata caching
- Chunking enabled

### Phase 3: Scale (1M - 100M users)

```
[Client Apps] → [Global CDN]
                     ↓
[Regional API Gateways]
                     ↓
[Microservices (100+ instances)]
                     ↓
[Cassandra (metadata)] + [Redis Cluster] + [Multi-region S3]
                     ↓
[Kafka (event streaming)]
```

- Cassandra for metadata (horizontal scaling)
- Deduplication enabled
- Multi-region storage
- Kafka for event streaming

### Phase 4: Massive Scale (100M+ users)

```
[Global Edge Network]
                     ↓
[Multi-region Architecture]
                     ↓
[1000+ Microservice Instances]
                     ↓
[Distributed Databases (multi-DC)]
                     ↓
[10 EB+ Storage across regions]
```

- Multi-region deployment
- Advanced deduplication
- ML-based conflict resolution
- Petabyte-scale storage per region

## Interview Talking Points

### Key Discussion Areas

**1. Critical Questions:**
- "How do you handle the same file edited offline on 2 devices?"
  - Conflict detection + keep both versions
- "How do you ensure data durability?"
  - Replication (3x) + erasure coding + multi-region
- "How do you optimize bandwidth for large files?"
  - Chunking + deduplication + delta sync

**2. System Bottlenecks:**
- "Metadata updates can become bottleneck"
  - Solution: Cassandra for horizontal scaling
- "Storage costs grow linearly with users"
  - Solution: Deduplication (30-50% savings)
- "Sync conflicts can frustrate users"
  - Solution: Three-way merge + conflict-free data types

**3. Optimizations:**
- "Use CDC chunking for better dedup"
- "Cache popular files at CDN edge"
- "Batch metadata updates"
- "Compress chunks before storage"

**4. Advanced Features:**
- "How would you add real-time collaboration?"
  - Operational Transform or CRDTs
- "How would you implement file search?"
  - Elasticsearch with file content indexing
- "How would you handle ransomware?"
  - Version history + point-in-time recovery

## Summary

Distributed file storage demonstrates:

**Core Challenges:**
1. **Sync Conflicts**: Resolve concurrent edits elegantly
2. **Data Durability**: Never lose user data (11 nines)
3. **Bandwidth Optimization**: Minimize upload/download
4. **Storage Efficiency**: Deduplication at scale
5. **Real-Time Sync**: Sub-second updates across devices

**Key Solutions:**
- Content-defined chunking for deduplication
- Delta sync (upload only changes)
- Keep-both conflict resolution
- Multi-tier caching (edge + regional + origin)
- Event-driven sync notifications
- Multi-region replication

**Scale Achievements:**
- 500M users
- 10 EB storage (10,000 PB)
- 30-50% storage savings via dedup
- <5 second sync latency
- 99.99% availability
- 99.999999999% durability

This design mirrors Dropbox, Google Drive, OneDrive implementations, proving that chunking, deduplication, and intelligent conflict resolution are critical for file storage at scale.
