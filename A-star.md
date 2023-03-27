~~~python
from heapq import heapify, heappush, heappop
import math
# Use case for Euclidean distance heuristic

# If the edges between nodes in a graph are straight lines, 
# then Euclidean distance is a valid heuristic and it will never over estimate the distance
# between any two nodes of the graph. 

# The heuristic distance can not over estimate the distance between a node in a graph
# and the goal node. The shortest distance between two points on a 2d plane is a straight line, and given two points
# on a 2d plane, the Euclidean distance is the magnitude of the straight line between these two points.

# As seen on the map diagram, the edges between the nodes are straight lines, this justifies using the Euclidean
# distance heuristic

# I found the provided notebook for Dijkstra's algorithm a good reference for solving this problem, A* is similar
# except there is more information provided from the heuristic distance from a node to the goal node
# which influences the sequence of explored nodes

# I looked at this wikipedia article to get ideas for this task https://en.wikipedia.org/wiki/Best-first_search

# Three distance functions with explanations

def manhattan_distance(x1, x2, y1, y2):
    
    # Manhattan distance can over estimate the distance between a node and the goal node
    # If nodes are connected diagonally, Manhattan distance will over estimate unless there
    # is the same x or y coordinates for the nodes
    # So the Manhattan distance metric will have to be adjusted to be a valid
    # heuristic

    # If there was no diagonal movement allowed for the graph then Manhattan distance would be a valid heuristic
    
    return abs(x2 - x1) + abs(y2 - y1)

def Chebyshev_distance(x1, x2, y1, y2):
    
    # Chebyshev_distance
    # https://en.wikipedia.org/wiki/Chebyshev_distance

    # Chebyshev_distance will always be less than or equal to both Manhattan distance
    # and Euclidean distance

    # This makes it a valid heuristic for graphs that allow for diagonal/angled movement and do not allow
    # diagonal/angled movement. Chebyshev distance can be used for this particular problem because it
    # will not over estimate the actual distance between a node and the goal node
    
    return max(abs(x2 - x1), abs(y2 - y1))

def euclidean_distance(x1, x2, y1, y2):
    
    # Euclidean distance will never over estimate the distance between two nodes in this graph
    # On a 2D plane, the shortest distance between two points is a straight line which is the Euclidean distance
    # The Euclidean distance is the magnitude of the hypotenuse of a right triangle
    
    return math.sqrt((y2 - y1) ** 2 + (x2 - x1) ** 2)
    
# distance functions
distance_functions = {'EUCLIDEAN': euclidean_distance,
                      'CHEBYSHEV': Chebyshev_distance,
                      'MANHATTAN': manhattan_distance}

# Easily change the heuristic function
HEURISTIC = 'EUCLIDEAN'

# distance function f
f = lambda node1, node2: distance_functions['EUCLIDEAN'](node1.x, node2.x, node1.y, node2.y)
g = lambda node1, node2: distance_functions[HEURISTIC](node1.x, node2.x, node1.y, node2.y)
class GraphNode:
    
    def __init__(self, value, x, y):
        self.value = value
        self.x = x
        self.y = y
        self.distance = math.inf
        self.heuristic = math.inf
        self.neighbors = []
        self.predecessor = None
        
    def add_neighbor(self, neighbor):
        self.neighbors.append(neighbor)
        
    def __lt__(self, node):
        return self.heuristic < node.heuristic
    
def get_path(node):
    
    path = []
    while node:
        path.append(node.value)
        node = node.predecessor
        
    path.reverse()
    return path
    

def shortest_path(M,start,goal):
    
    # add all nodes to list
    nodes = [GraphNode(i, M.intersections[i][0], M.intersections[i][1]) for i in range(len(M.intersections))]
    
    # add all neighbors to the nodes
    for i in range(len(M.roads)):
        roads = M.roads[i]
        for road in roads:
            nodes[i].add_neighbor(nodes[road])

    goal_node = nodes[goal]
    # Heuristic is Euclidean distance between a node and the goal node, this will be <= to the actual distance
    heuristic = {node.value: g(node, goal_node) for node in nodes}

    # create the min queue
    queue = []
    heapify(queue)
    nodes[start].distance = 0
    nodes[start].heuristic = heuristic[nodes[start].value]
    
    # push start node into min queue
    heappush(queue, nodes[start])
    

    # seen nodes, minimum path distance, and path of node value
    seen = set()
    min_path = math.inf
    path = []
    
    while len(queue) > 0:
        
        current_node = heappop(queue)

        for neighbor in current_node.neighbors:
            if neighbor not in seen:
                if neighbor == goal_node:
                    distance = current_node.distance + f(current_node, neighbor)
                    if distance < min_path:
                        min_path = distance
                        neighbor.distance = distance
                        neighbor.predecessor = current_node
                        return get_path(goal_node)

                else:
                    if current_node.distance + f(current_node, neighbor) + heuristic[neighbor.value] < neighbor.heuristic:
                        neighbor.distance = current_node.distance + f(current_node, neighbor)
                        neighbor.heuristic = current_node.distance + f(current_node, neighbor) + heuristic[neighbor.value]
                        neighbor.predecessor = current_node
                        heappush(queue, neighbor)
                        seen.add(current_node)
                    
                    
             
    print("shortest path called")
    return [nodes[start].value]
    
~~~
