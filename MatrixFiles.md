# MatrixFiles

MatrixFiles will allow you to store files in Matrix. Each file is represented by a room and can be modified by anyone who is allowed to send state events to that room.

### m.room.room_type

The m.room.room_type event from MSC1840 is extended with two new room types.

*state_key*: An empty string

|Content Key|Type|Description|
|-|-|-|
|type|string|One of "c.room_type.file", "c.room_type.directory".|

## Directories

A directory contains a c.directory.entry state event for each file or subdirectory. The special links "." and ".." are not stored.

### c.directory.entry

*state_key*: The filename.

|Content Key|Type|Description|
|-|-|-|
|room_id|string|The room id where this file or subdirectory is located. If this is missing the event should be ignored.|
|mtime|int|The modification timestamp.|

## Files

Each file contains a number of c.file.data_block events. To read a file traverse all these events ordered by their index. The data should be uploaded with a mimetype of "application/octet-stream". 

### c.file.data_block

*state_key*: The byte index of this data block. The first data block will always have an index of 0.

|Content Key|Type|Description|
|-|-|-|
|url|string|An mxid specifying where the data is located. If this is missing the event should be ignored.|
|size|int|The data size in bytes. If this is missing the event should be ignored.|
|mtime|int|The modification timestamp.|

### c.file.chunk

*state_key*: The state key of this chunk. The first chunk will have a state key of an empty string.

|Content Key|Type|Description|
|-|-|-|
|url|string|An mxid specifying where the data is located. If this is missing the event should be treated is if it was empty.|
|size|int|The data size in bytes. If this is missing the event should be treated is if it was empty.|
|next|string|The state_key of the next chunk. If this is missing the event should be treated is if it was empty. If this points to a state_key that has already been visited, the file is corrupt, since that would result in an infinitely large file.|
|mtime|int|The modification timestamp.|

## Timestamps

Files and directories have two different timestamps, called ctime (change time) and mtime (modification time). Each timestamp has a precision of one millisecond.

||ctime|mtime|
|-|-|-|
|File|The highest homeserver timestamp in any c.file.data_block event.|The highest value of mtime in the c.file.data_block events.|
|Directory|The highest homeserver timestamp in any c.directory.entry event.|The highest value of mtime in the c.directory.entry events.|
|Symlink|||


## Security considerations

Since a filename can contain any unicode character, a implementation should be prepared to handle maliciously generated filenames. This could be by escaping the filename or ignoring that file.

A directory can link to itself or to one of its parent directories. If a client tries to recursively traverse such a directory structure it may never reach the end.

By linking multiple times to the same data (both in files and directories) it is possible to create huge structures using only very little storage.

A malicious homeserver could respond with different data for the same mxid. This could result in different clients getting different data for the same file.


## Symlinks

### c.symlink.destination

*state_key*: An empty string

|Content Key|Type|Description|
|-|-|-|
|destination|string|**Required.** Which file this symlink links to.|


