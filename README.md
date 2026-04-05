# Roblox-Remote-Batch
Per-frame remote event batching system for Roblox games, with tagged dispatch, client-specific sends, and smart batch merging.

This module is a Luau networking utility for Roblox that reduces remote event spam by consolidating data into tagged batches sent once per frame through a single remote. Instead of firing many separate remotes for every gameplay update, systems can add data to a shared batch and let the module dispatch it efficiently between server and clients.

The module supports broadcast batching, single-client batching, and tag-based listeners for handling incoming data cleanly. It also includes smart batch merge helpers that can combine new data into existing batch entries using nested search keys, which is useful for reducing duplicate updates and keeping batched payloads organized.

This system is designed for multiplayer games that need cleaner network architecture, lower remote call volume, and a more scalable way to process frequent state updates.
