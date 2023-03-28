# Huffman Encoding and Decoding

**Time Complexity**: $O(n log \ n)$, where n is the number of characters of the input string.

**Space Complexity**: $O(n)$, where n is the number of characters of the input string.
___
The input for the `huffman_encoding` function is a string with length $\geq 1$. The outputs are an encoded string, and a tree data structure. The tree can then decode the encoded string, the decoding function produces the original input string. Some crititcal data structures for encoding and decoding include nodes, trees, dictionaries, and a min heap.

### Getting letter frequencies
A dictionary storing the frequecy of each letter is constructed by reading each letter in the string and incrementing its count in a dictionary. This process is computed in time linear to the length of the string, $O(n)$. Where $n$ is the length of the input string. To handle the edge case of the input string being composed of a single character, I added another key value pair to the dictionary so a tree can be constructed. I saw on https://cboard.cprogramming.com/cplusplus-programming/56414-huffman-code-single-character-file.html the idea of 'dummy node' so I incorporated this extra node just for the purpose of constructing a tree.

### Creating A Min Heap of Nodes
For each key in the letter frequency dictionary, a Node is created that contains the letter, and the frequency of the letter. There are other properties of the Node, `self.left` and `self.right` that point to left and right child nodes. Finally each node has a property `self.bit`, that stores either a $1$ or a $0$. The node also has a method `__gt__(self, other_node)` that is used to compare one node to another. This method is important to store the nodes in the proper order in the min heap. Adding an element to a min heap is done in $O(log \ n)$ time, since there are n nodes, adding the nodes to the heap is done in $O(nlog \ n)$ time.

### Creating a tree
While there are at least two nodes in the heap, remove the two nodes from the heap and create a new node using the merge function. The merge function takes two nodes as input, and outputs a parent node with the two input nodes as child nodes. The child nodes are assigned bit values, which are determined by their frequency property. The node with the higher frequency is assigned a $1$ and the node with the lower frequency is assigned a $0$. Also, the node with higher frequency is the right child and the node with lower frequency is the left child. The frequency property of the parent node is the sum of the frequencies of the child nodes. The tree construction process completes when there is only one element in the min heap, which is the root of the tree. This is stored in the `tree` variable. The time complexity of this process is $O(nlog \ n)$ due to the addition and removal of nodes from the min heap.

### Creating codes for each character
To produce a dictionary of key value pairs where the key is the letter and the code is the value, all child nodes of the constructed tree must be visited. Since there are n distinct characters in the string, and it takes on average logn steps to reach a leaf, this process has time complexity of $O(nlog \ n)$. The path to the leaf nodes determines the value for the key.

### Encoding the string
To encode the string, initialize an empty string and concatenate the code for each character to this string. This is done in $O(n)$ time where n is the number of characters in the input string. Reading a value from the dictionary is done in constant time.


### Decoding the string
To decode the encoded string, read the bit at the zeroth index of the encoded string and move left in the tree if the bit is $0$, and move right if the bit is 1. If the current node is not a leaf remove the $0^{th}$ bit from the encoded string. If the current node is a leaf concatenate the leaf value to `data` and continue this process until the encoded string is empty. Once the encoded string is empty return the string `data`. The process of decoding the encoded string is proportional to the length n of the encoded string. For each bit of the encoded string, only constant time computations are done, making the time complexity of this process $O(n)$.

The space complexity is linearly proportional to the length of the input string. The input string converts to an encoded string that is eventually decoded that is proportional in size to the input string.

~~~python
import sys
from heapq import heapify, heappush, heappop
from collections import defaultdict


class Node:
    
    def __init__(self, frequency, value = None):
        self.value = value
        self.frequency = frequency
        self.left = None
        self.right = None
        self.bit = None
        
    def __gt__(self, other_node):
        return self.frequency > other_node.frequency
        
def merge_nodes(node1, node2):
    
    combined_frequency = node1.frequency + node2.frequency
    parent = Node(combined_frequency)
    if node1.frequency < node2.frequency:
        parent.left = node1
        parent.right = node2
        node1.bit = 0
        node2.bit = 1
    else:
        parent.left = node2
        parent.right = node1
        node2.bit = 1
        node1.bit = 0
        
    return parent

def get_frequencies(data):
    
    assert type(data) == str

    frequencies = defaultdict(int)
    for val in data:
        frequencies[val] += 1
        
    if len(frequencies.keys()) == 1:
        #https://cboard.cprogramming.com/cplusplus-programming/56414-huffman-code-single-character-file.html
        # Idea for dummy node
        frequencies['filler'] = 1
    return frequencies


def huffman_encoding(data):
    
    assert(len(data) > 0)
    def create_tree(data):
            
        frequencies = get_frequencies(data)
        min_heap = [Node(frequencies[key], key) for key in frequencies]
        heapify(min_heap)
        while len(min_heap) > 1:
            node1 = heappop(min_heap)
            node2 = heappop(min_heap)
            new_node = merge_nodes(node1, node2)
            heappush(min_heap, new_node)
            
        tree =  heappop(min_heap)
        
        return tree
    
    def get_codes(tree, route='', routes = {}):
        
        
        if tree.value:
            routes[tree.value] = route
        else:
            temp1 = route + '0'
            temp2 = route + '1'
            get_codes(tree.left, temp1)
            get_codes(tree.right, temp2)
            
        return routes
    

    def encode_string(codes, data):
        
        encoded_string = ''
        for char in data:
            encoded_string += codes[char]
            
        return encoded_string
    
    tree = create_tree(data)
    codes = get_codes(tree)

    encoded_string = encode_string(codes, data)
    return encoded_string, tree


def huffman_decoding(data,tree):
    
    huffman_tree = [tree]
    current_tree = huffman_tree[0]
    output = ''
    
    while True:

        if data == '':
            output += current_tree.value
            break
        if current_tree.value != None:
            output += current_tree.value
            current_tree = huffman_tree[0]
        elif data[0] == '0':
            current_tree = current_tree.left
            data = data[1:]
        elif data[0] == '1':
            current_tree = current_tree.right
            data = data[1:]
            
    return output
