# LocalSharedMemory

**A lightweight, developer-friendly memory manager for `WebAssembly.Memory` and `SharedArrayBuffer` in JavaScript.**

`LocalSharedMemory` extends the standard `WebAssembly.Memory` object, equipping it with a familiar C-style `malloc` allocation system and a rich set of tools for easier, safer, and more intuitive memory manipulation in multi-threaded web environments.

It's designed to simplify the complexities of manual memory management when working with Workers, providing a structured and efficient way to allocate and access shared data blocks.

## Key Features

- **C-Style Memory Allocation:** Simple and powerful `malloc()` for dynamic memory allocation.
- **Automatic Memory Alignment:** Ensures allocated blocks meet alignment requirements for optimal performance.
- **Easy Allocation Tracking:** Inspect all allocated memory blocks with the `allocs()` method.
- **Rich Data Accessors:** Includes a comprehensive set of `get/set` methods for all standard numeric types (e.g., `getUint32`, `setFloat64`).
- **Typed Array Views:** Easily create `TypedArray` views (`Uint8Array`, `Float32Array`, etc.) on your allocated memory segments.
- **Atomic Operations:** Built-in support for atomic operations for safe, concurrent data manipulation in multi-threaded applications.
- **Seamless Integration:** As an extension of `WebAssembly.Memory`, it can be used wherever a standard memory object is expected.

## Quick Start

Here's a simple example of how to use `LocalSharedMemory`:

```javascript
import LocalSharedMemory from "./LocalSharedMemory.js";

// 1. Create a new shared memory instance.
const memory = new LocalSharedMemory();

// 2. Allocate memory.
// Allocate 24 bytes, aligned to the default 16 bytes.
const ptr1 = memory.malloc(24);

// Allocate 5 bytes, aligned to 2 bytes.
const ptr2 = memory.malloc(5, 2); 

// 3. Write and read data.
// Set a 32-bit integer at the beginning of the first allocation.
memory.setInt32(ptr1, 12345);

// Get the value back.
const myValue = memory.getInt32(ptr1); // Returns 12345

// 4. Create a TypedArray view on the second allocation.
const myData = new Uint8Array([10, 20, 30, 40, 50]);
const uint8View = memory.arrayView(Uint8Array, ptr2);
uint8View.set(myData);

// 5. Inspect all allocations.
console.table(memory.allocs());
```

## API Reference

### `new LocalSharedMemory(initial, maximum, shared)`

Creates a new memory instance. It accepts the same parameters as the standard `WebAssembly.Memory` constructor.

-   `initial` (Number): The initial size of the memory, in WebAssembly pages (64KB each). Defaults to `1000`.
-   `maximum` (Number): The maximum size the memory is allowed to grow to. Defaults to `initial`.
-   `shared` (Boolean): Whether the memory is shared. Defaults to `true`.

---

### `malloc(byteLength, alignBytes = 16)`

Allocates a block of memory.

-   `byteLength` (Number): The required size of the allocation in bytes.
-   `alignBytes` (Number): The byte boundary to align the allocation to. Defaults to `16`.
-   **Returns** (Number): The byte offset (pointer) to the start of the allocated block.

Each allocation includes an 8-byte header just before the returned offset, which stores the requested `byteLength` and the total internal buffer size.

---

### `allocs()`

Retrieves a list of all memory blocks allocated by `malloc()`.

-   **Returns** (Array): An array of objects, where each object contains information about an allocation:
    -   `byteOffset`: The starting offset of the writable data area.
    -   `byteLength`: The user-requested size of the allocation.
    -   `bufferSize`: The total space used by the allocation in the buffer (including headers and alignment padding).
    -   `buffer`: A `SharedArrayBuffer` slice of the allocated block for easy debugging.

---

### `sizeof(offset)`

Returns the user-requested `byteLength` of an allocation at a given offset.

-   `offset` (Number): The pointer returned by `malloc()`.
-   **Returns** (Number): The size of the allocation in bytes.

```javascript
const byteOffset = memory.malloc(100);
console.log(memory.sizeof(byteOffset)); // 100
```

---

### Data Accessors & Views

`LocalSharedMemory` provides a rich API for accessing and manipulating the data in the buffer.

#### View Methods

-   `arrayBuffer(offset, length)`: Returns a sliced `ArrayBuffer` copy (not shared). Useful for APIs that don't accept shared buffers.
-   `sharedArrayBuffer(offset, length)`: Returns a sliced `SharedArrayBuffer` view.
-   `dataView(offset, length)`: Returns a `DataView` of a memory segment.
-   `arrayView(TypedArray, offset, length)`: Returns a `TypedArray` (e.g., `Uint8Array`, `Float32Array`) view of a memory segment.

#### Getter/Setter Methods

A full suite of getter and setter methods are available for all standard numeric types. They all follow the same pattern:

-   `get<Type>(offset)`
-   `set<Type>(offset, value)`

**Example:**

```javascript
memory.setFloat64(ptr1, Math.PI);
const pi = memory.getFloat64(ptr1);
```

**Available Types:** `Uint8`, `Int8`, `Uint16`, `Int16`, `Uint32`, `Int32`, `BigUint64`, `BigInt64`, `Float32`, `Float64`.

---

### Atomic Operations

For safe concurrent operations in multi-threaded environments, you can use the built-in atomic methods, which wrap the global `Atomics` object.

-   `add(TypedArray, index, value)`: Atomically adds a value at a given index.
-   ... and other standard atomic operations like `sub`, `and`, `or`, `xor`, `exchange`, `compareExchange`, `load`, `store`.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.