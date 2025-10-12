# LocalSharedMemory #
WebAssembly Memory but a few skills

## Purpose of Script ##
LocalSharedMemory is a simple WebAssembly.Memory object which can be transferred between workers but it has some abilities:
#
### 1. malloc() ###
    It is regular C like malloc function that can allocate space in memory. First two unsigned integers (8 bytes) of LocalSharedMemory is used by total allocation information and every malloc request will stored here:
    
    const byteLength = new Uint32Array( bufferOfLocalSharedMemory )[0]
    const allocCount = new Uint32Array( bufferOfLocalSharedMemory )[1]

    malloc function can call with two parameters:
    
    @param {Number} byteLength - total requrired size of allocation
    @param {Number} alignBytes - to align start of allocated space (optional, default: 16) 

    @return {Number} byte offset for you

    For example, calling function with "const ptr = malloc(1)" code (probably) returns 16 (or multiple of 16) and this number can be used to store data. Next allocation requests will be multiples of 16 too. 
    
    Example usages:
    /* aligned with 16 bytes */
    const byteOffset = memory.malloc(7)   
    const uInt8Array = new Uint8Array(memory.buffer, byteOffset, 7)

    /* aligned with 2 btyes  */
    const byteOffset = memory.malloc(5, 2)     
    const uInt8Array = new Uint8Array(memory.buffer, byteOffset, 5)

    Every request will be sum with 8 bytes which is header information of your request. For each allocation uses two headers that stores just before your byte offset (byteOffset - 8). Those headers are:

    1. byteLength (byteOffset - 8): requested allocation size
    2. bufferSize (byteOffset - 4): used size (align + headers + size)    

    Headers are used to find previous allocations and loop over them which can be easy found with next title: allocs()

#

### 2. allocs() ###
    Previous allocations can be gathered with calling this function. Regular array will be return that holds objects which every one of them stores allocation information:

    - buffer        : data buffer (sliced for easy debug)
    - byteOffset    : writable start offset
    - byteLength    : requested space of user
    - bufferSize    : total used space (headers + align + request)

    This function can be used to see allocations.

#

### 3. sizeof() ###
    When you get your byteOffset with a malloc() request then you can use sizeof() function to get byteLength of byteOffset:

    const byteLength = 5;
    const byteOffset = memory.malloc(byteLength);

    const size = memory.sizeof( byteOffset );
    console.log( size ); /* prints 5 into console */

#

### 4. arrayBuffer() ###
    If you want to slice your allocation to an ArrayBuffer then you can call this function. This function can be used two parameters:

    @param {Number} byteOffset - offset with gathered by malloc()
    @param {Number} byteLength - (optional, default: sizeof(byteOffset))

    @return {ArrayBuffer} - sliced part from SharedArrayBuffer

    Regular slice calls over memory.buffer returns instance of SharedArrayBuffer which can NOT be used in non-transferable daha channels (such as BroadcastChannel).

    Slicing shared buffer is NOT contains "set" calls because of using slice method over TypedArrays returns already ArrayBuffer instances instead of slicing buffer directly.  