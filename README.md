# Class to represent a memory block
class MemoryBlock:
    def __init__(self, start, size, is_free=True):
        self.start = start        # Starting address of the memory block
        self.size = size          # Size of the memory block
        self.is_free = is_free    # Whether the block is free or allocated
        self.next = None          # Pointer to the next block (linked list structure)
# Function to allocate memory using the Best Fit algorithm
def allocate_memory(head, request_size):
    """
    Finds the smallest free block that fits the requested size.
    If a block is found, it allocates memory and splits the block if needed.
    """
    current = head
    best_fit = None

    # Traverse the free list to find the best fit block
    while current:
        if current.is_free and current.size >= request_size:
            if not best_fit or current.size < best_fit.size:
                best_fit = current
        current = current.next

    if best_fit:
        # Split the block if there is leftover space
        remaining_size = best_fit.size - request_size
        best_fit.size = request_size
        best_fit.is_free = False

        if remaining_size > 0:
            new_block = MemoryBlock(best_fit.start + request_size, remaining_size)
            new_block.next = best_fit.next
            best_fit.next = new_block

        return best_fit.start  # Return starting address of allocated block
    else:
        print("Error: Insufficient memory.")
        return -1  # Return error code if allocation fails
# Function to deallocate a memory block
def deallocate_memory(head, start_address):
    """
    Frees a memory block by its starting address.
    Merges adjacent free blocks to reduce fragmentation.
    """
    current = head

    # Find the block with the given starting address
    while current:
        if current.start == start_address and not current.is_free:
            current.is_free = True

            # Merge with the next block if it's free
            if current.next and current.next.is_free:
                current.size += current.next.size
                current.next = current.next.next

            return True
        current = current.next

    print("Error: Invalid deallocation request.")
    return False  # Return False if block not found or already free
# Function to display the current memory allocation status
def display_status(head):
    """
    Prints a table of all memory blocks, showing their start address, size, and status.
    """
    print(f"{'Block ID':<10}{'Start Address':<15}{'Size':<10}{'Status':<10}")
    block_id = 0
    current = head

    while current:
        status = "Free" if current.is_free else "Allocated"
        print(f"{block_id:<10}{current.start:<15}{current.size:<10}{status:<10}")
        block_id += 1
        current = current.next
# Function to analyze external fragmentation
def fragmentation_analysis(head, request_size):
    """
    Calculates the total external fragmentation, which is the sum of free block sizes
    that are smaller than the requested size.
    """
    external_fragments = 0
    current = head

    while current:
        if current.is_free and current.size < request_size:
            external_fragments += current.size
        current = current.next

    print(f"External Fragmentation: {external_fragments} units")
# CLI to interact with the memory management system
def cli_interface(head):
    """
    Provides a command-line interface for memory allocation, deallocation,
    and displaying memory status.
    """
    print("Memory Management CLI")
    print("Commands: allocate <size>, deallocate <start_address>, status, fragmentation <size>, exit")
    while True:
        command = input("Enter command: ").strip().split()

        if not command:
            continue

        if command[0] == "allocate" and len(command) == 2:
            try:
                size = int(command[1])
                allocate_memory(head, size)
            except ValueError:
                print("Error: Invalid size.")

        elif command[0] == "deallocate" and len(command) == 2:
            try:
                start_address = int(command[1])
                deallocate_memory(head, start_address)
            except ValueError:
                print("Error: Invalid address.")

        elif command[0] == "status":
            display_status(head)

        elif command[0] == "fragmentation" and len(command) == 2:
            try:
                size = int(command[1])
                fragmentation_analysis(head, size)
            except ValueError:
                print("Error: Invalid size.")

        elif command[0] == "exit":
            break

        else:
            print("Invalid command. Use: allocate, deallocate, status, fragmentation, exit.")
# Function to test the Best Fit memory management system
def test_best_fit():
    """
    Performs various allocation and deallocation operations to test the system's functionality.
    """
    head = MemoryBlock(0, 1000)  # Initialize memory with 1000 units
    print("Initial memory state:")
    display_status(head)

    print("\nAllocating 100 units:")
    allocate_memory(head, 100)
    display_status(head)

    print("\nAllocating 200 units:")
    allocate_memory(head, 200)
    display_status(head)

    print("\nDeallocating the first block (100 units):")
    deallocate_memory(head, 0)
    display_status(head)

    print("\nFragmentation Analysis for request size 150:")
    fragmentation_analysis(head, 150)
if __name__ == "__main__":
    # Initialize memory with a single block of 1000 units
    head = MemoryBlock(0, 1000)
    cli_interface(head)
