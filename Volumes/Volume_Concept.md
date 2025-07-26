By default, Docker containers use a writable layer for data storage, which is ephemeral. This means any data written during runtime is lost when the container is removed, as the writable layer is deleted along with the container. To persist data, Docker provides volumes and bind mounts that exist independently of the container lifecycle.
On Windows, Docker typically runs inside a Linux virtual machine using WSL 2 (Windows Subsystem for Linux) or Hyper-V, depending on your setup. The writable layer of a container is stored inside this VM, not directly on the Windows filesystem.

🔍 Writable Layer Location on Windows (WSL 2)
If you're using WSL 2, Docker stores container data in a virtual disk file located at:

C:\Users\<YourUsername>\AppData\Local\Docker\wsl\data\ext4.vhdx

This .vhdx file is a virtual Linux filesystem that contains all Docker data, including:

Image layers
Writable layers
Volumes
Metadata

Default Docker Storage Location on Linux
Docker stores all container data (including writable layers, images, volumes, etc.) under:

/var/lib/docker/

Directory	       Purpose
overlay2/	     Stores image layers and container writable layers (if     using the overlay2 driver)
containers/  	Metadata and logs for each container
volumes/	    Docker-managed volumes
image/	        Image metadata
network/    	Network configuration

1) What Is a Host Volume (Bind Mount)?
It maps a specific path on the host to a path inside the container.
Changes made inside the container are reflected on the host, and vice versa.
Useful for:
Sharing configuration files
Persisting logs or data
Live code development

docker run -dit --name web01 -v C:\Users\v-mre\k8:/host_vol nginx


2) Anonymous Volume
An anonymous volume is a Docker-managed volume that:

Is created automatically when you mount a volume without specifying a name.
Is stored in Docker’s internal volume storage (usually under /var/lib/docker/volumes on Linux).
Is persistent across container restarts but not easily identifiable unless you inspect the container.
Is shared between the host and container if mounted, but not directly visible in your local filesystem unless you inspect or mount it manually.
Example:

This creates an anonymous volume and mounts it to /data inside the container.

✏️ Writable Layer
Every Docker container has a writable layer on top of its image:

This layer is temporary and unique to the container.
Any changes made (like creating files, installing packages) are stored here.
When the container is deleted, this layer is also deleted—changes are lost unless persisted via volumes.
Example:
If you run:


And then inside the container:


That file is stored in the writable layer. If you remove the container, the file is gone.

🔍 Key Differences
Feature	Anonymous Volume	Writable Layer
Persistence	Yes (until volume is removed)	No (deleted with container)
Visibility	Not directly visible in host FS	Not visible in host FS
Purpose	Store persistent data	Temporary container changes
Usage	Mounted explicitly or implicitly	Always present in every container


 Named Volumes in Docker
A named volume is a persistent storage mechanism managed by Docker that allows you to store data outside the container's writable layer. Unlike anonymous volumes, named volumes have a user-defined name, making them easier to manage, reuse, and inspect.

✅ How to Create and Use a Named Volume
You can create and use a named volume like this:
docker volume create mydata
docker run -dit --name web01 -v mydata:/host_vol nginx


This mounts the named volume mydata to /host_vol inside the container.

🔍 Benefits of Named Volumes
Feature	Named Volume
Persistence	Yes, survives container deletion
Reusability	Can be reused across containers
Manageability	Easy to inspect, backup, and remove
Isolation	Isolated from host filesystem
Location	Stored in Docker's volume directory
📦 Volume Commands
List volumes:
docker volume ls


Inspect a volume:

docker volume inspect mydata

Remove a volume:

docker volume rm mydata




Host Volume vs Named Volume
===========================
Feature	Host Volume	Named Volume
Definition	Mounts a specific directory from the host filesystem	A Docker-managed volume stored in Docker's volume storage
Syntax	-v /path/on/host:/path/in/container	-v volume_name:/path/in/container
Location	Anywhere on the host (e.g., C:\data, /home/user)	Usually under /var/lib/docker/volumes
Persistence	Persists as long as the host directory exists	Persists until explicitly removed via Docker
Visibility	Directly visible and editable from the host OS	Not directly visible unless inspected via Docker
Use Case	When you want to work with existing host files	When you want Docker to manage the storage lifecycle
Portability	Less portable (depends on host path)	More portable (Docker manages location)
Security	Can expose sensitive host paths	Isolated from host filesystem
Management	Managed manually via OS	Managed via Docker CLI (docker volume)


🧠 What is tmpfs?
tmpfs is a temporary filesystem stored in memory (RAM). It’s fast and volatile—meaning data stored in it disappears after a reboot or container stop.

🐳 tmpfs in Docker
In Docker, you can mount a tmpfs volume like this:
docker run -dit --name mycontainer --tmpfs /mytmpfs nginx


This creates a memory-backed filesystem at /mytmpfs inside the container.

🔍 Use Cases
Storing sensitive data temporarily
High-speed read/write operations
Avoiding disk I/O for ephemeral data