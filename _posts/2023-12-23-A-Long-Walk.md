---
title: A Long Walk
description: Advent of Code 2023 [Day 23]
layout: default
lang: en
tag: aoc23
prefetch:
  - adventofcode.com
---

The Elves resume water filtering operations! Clean water starts flowing over the edge of Island Island.

They offer to help **you** go over the edge of Island Island, too! Just hold on tight to one end of this impossibly long rope and they'll lower you down a safe distance from the massive waterfall you just created.

As you finally reach Snow Island, you see that the water isn't really reaching the ground: it's being **absorbed by the air** itself. It looks like you'll finally have a little downtime while the moisture builds up to snow-producing levels. Snow Island is pretty scenic, even without any snow; why not take a walk?

There's a map of nearby hiking trails (your puzzle input) that indicates **paths** (`.`), **forest** (`#`), and steep **slopes** (`^`, `>`, `v`, and `<`).

For example:

```
#.#####################
#.......#########...###
#######.#########.#.###
###.....#.>.>.###.#.###
###v#####.#v#.###.#.###
###.>...#.#.#.....#...#
###v###.#.#.#########.#
###...#.#.#.......#...#
#####.#.#.#######.#.###
#.....#.#.#.......#...#
#.#####.#.#.#########v#
#.#...#...#...###...>.#
#.#.#v#######v###.###v#
#...#.>.#...>.>.#.###.#
#####v#.#.###v#.#.###.#
#.....#...#...#.#.#...#
#.#########.###.#.#.###
#...###...#...#...#.###
###.###.#.###v#####v###
#...#...#.#.>.>.#.>.###
#.###.###.#.###.#.#v###
#.....###...###...#...#
#####################.#
```

You're currently on the single path tile in the top row; your goal is to reach the single path tile in the bottom row. Because of all the mist from the waterfall, the slopes are probably quite **icy**; if you step onto a slope tile, your next step must be **downhill** (in the direction the arrow is pointing). To make sure you have the most scenic hike possible, **never step onto the same tile twice**. What is the longest hike you can take?

In the example above, the longest hike you can take is marked with `O`, and your starting position is marked `S`:

```
#S#####################
#OOOOOOO#########...###
#######O#########.#.###
###OOOOO#OOO>.###.#.###
###O#####O#O#.###.#.###
###OOOOO#O#O#.....#...#
###v###O#O#O#########.#
###...#O#O#OOOOOOO#...#
#####.#O#O#######O#.###
#.....#O#O#OOOOOOO#...#
#.#####O#O#O#########v#
#.#...#OOO#OOO###OOOOO#
#.#.#v#######O###O###O#
#...#.>.#...>OOO#O###O#
#####v#.#.###v#O#O###O#
#.....#...#...#O#O#OOO#
#.#########.###O#O#O###
#...###...#...#OOO#O###
###.###.#.###v#####O###
#...#...#.#.>.>.#.>O###
#.###.###.#.###.#.#O###
#.....###...###...#OOO#
#####################O#
```

This hike contains `94` steps. (The other possible hikes you could have taken were `90`, `86`, `82`, `82`, and `74` steps long.)

Find the longest hike you can take through the hiking trails listed on your map. **How many steps long is the longest hike?**

```go
type Pos struct {
	row, col int
}

func readInput(filename string) [][]byte {
	data, err := os.ReadFile(filename)
	if err != nil {
		return [][]byte{}
	}
	lines := strings.Split(strings.TrimSpace(string(data)), "\n")
	grid := make([][]byte, len(lines))
	for i, line := range lines {
		grid[i] = []byte(line)
	}
	return grid
}

func isValid(grid [][]byte, pos Pos, visited map[Pos]bool) bool {
	return pos.row >= 0 && pos.row < len(grid) &&
		pos.col >= 0 && pos.col < len(grid[0]) &&
		grid[pos.row][pos.col] != '#' &&
		!visited[pos]
}

func dfs(grid [][]byte, pos Pos, end Pos, visited map[Pos]bool) int {
	if pos == end {
		return 0
	}

	maxSteps := -1
	visited[pos] = true
	defer delete(visited, pos)

	moves := [][2]int{ {-1, 0}, {1, 0}, {0, -1}, {0, 1} }
	curr := grid[pos.row][pos.col]

	// Handle slopes
	if curr == '>' {
		moves = [][2]int{ {0, 1} }
	} else if curr == '<' {
		moves = [][2]int{ {0, -1} }
	} else if curr == '^' {
		moves = [][2]int{ {-1, 0} }
	} else if curr == 'v' {
		moves = [][2]int{ {1, 0} }
	}

	for _, move := range moves {
		next := Pos{pos.row + move[0], pos.col + move[1]}
		if isValid(grid, next, visited) {
			if steps := dfs(grid, next, end, visited); steps >= 0 {
				if maxSteps < steps+1 {
					maxSteps = steps + 1
				}
			}
		}
	}

	return maxSteps
}

func findLongestHike(grid [][]byte) int {
	start := Pos{0, 1}
	end := Pos{len(grid) - 1, len(grid[0]) - 2}

	visited := make(map[Pos]bool)
	return dfs(grid, start, end, visited)
}

func main() {
	grid := readInput("input.txt")
	result := findLongestHike(grid)
	fmt.Println("Longest hike length:", result)
}
```

As you reach the trailhead, you realize that the ground isn't as slippery as you expected; you'll have **no problem** climbing up the steep slopes.

Now, treat all **slopes** as if they were normal **paths** (`.`). You still want to make sure you have the most scenic hike possible, so continue to ensure that you **never step onto the same tile twice**. What is the longest hike you can take?

In the example above, this increases the longest hike to `154` steps:

```
#S#####################
#OOOOOOO#########OOO###
#######O#########O#O###
###OOOOO#.>OOO###O#O###
###O#####.#O#O###O#O###
###O>...#.#O#OOOOO#OOO#
###O###.#.#O#########O#
###OOO#.#.#OOOOOOO#OOO#
#####O#.#.#######O#O###
#OOOOO#.#.#OOOOOOO#OOO#
#O#####.#.#O#########O#
#O#OOO#...#OOO###...>O#
#O#O#O#######O###.###O#
#OOO#O>.#...>O>.#.###O#
#####O#.#.###O#.#.###O#
#OOOOO#...#OOO#.#.#OOO#
#O#########O###.#.#O###
#OOO###OOO#OOO#...#O###
###O###O#O###O#####O###
#OOO#OOO#O#OOO>.#.>O###
#O###O###O#O###.#.#O###
#OOOOO###OOO###...#OOO#
#####################O#
```

Find the longest hike you can take through the surprisingly dry hiking trails listed on your map. **How many steps long is the longest hike?**

```go
type Pos struct {
	row, col int
}

type Edge struct {
	to     Pos
	length int
}

func readInput(filename string) [][]byte {
	data, err := os.ReadFile(filename)
	if err != nil {
		return [][]byte{}
	}
	return bytes2grid(data)
}

func bytes2grid(data []byte) [][]byte {
	lines := strings.Split(strings.TrimSpace(string(data)), "\n")
	grid := make([][]byte, len(lines))
	for i, line := range lines {
		grid[i] = []byte(line)
	}
	return grid
}

func findJunctions(grid [][]byte) []Pos {
	junctions := []Pos{}
	start := Pos{0, 1}
	end := Pos{len(grid) - 1, len(grid[0]) - 2}
	junctions = append(junctions, start, end)

	for r := 0; r < len(grid); r++ {
		for c := 0; c < len(grid[0]); c++ {
			if grid[r][c] == '#' {
				continue
			}
			exits := 0
			for _, d := range [][2]int{ {0, 1}, {0, -1}, {1, 0}, {-1, 0} } {
				nr, nc := r+d[0], c+d[1]
				if nr >= 0 && nr < len(grid) &&
					nc >= 0 && nc < len(grid[0]) &&
					grid[nr][nc] != '#' {
					exits++
				}
			}
			if exits > 2 {
				junctions = append(junctions, Pos{r, c})
			}
		}
	}
	return junctions
}

func buildGraph(grid [][]byte, junctions []Pos) map[Pos][]Edge {
	graph := make(map[Pos][]Edge)
	moves := [][2]int{ {0, 1}, {0, -1}, {1, 0}, {-1, 0} }

	for _, j := range junctions {
		visited := make(map[Pos]bool)
		stack := []struct {
			pos    Pos
			length int
		}{ {j, 0} }
		visited[j] = true

		for len(stack) > 0 {
			curr := stack[len(stack)-1]
			stack = stack[:len(stack)-1]

			if curr.length != 0 {
				for _, other := range junctions {
					if curr.pos == other {
						graph[j] = append(graph[j], Edge{other, curr.length})
						goto nextPath
					}
				}
			}

			for _, m := range moves {
				next := Pos{curr.pos.row + m[0], curr.pos.col + m[1]}
				if next.row >= 0 && next.row < len(grid) &&
					next.col >= 0 && next.col < len(grid[0]) &&
					grid[next.row][next.col] != '#' &&
					!visited[next] {
					visited[next] = true
					stack = append(stack, struct {
						pos    Pos
						length int
					}{next, curr.length + 1})
				}
			}
		nextPath:
		}
	}
	return graph
}

func findLongestPath(graph map[Pos][]Edge, start, end Pos, visited map[Pos]bool) int {
	if start == end {
		return 0
	}

	maxLength := -1
	visited[start] = true
	defer delete(visited, start)

	for _, edge := range graph[start] {
		if !visited[edge.to] {
			if length := findLongestPath(graph, edge.to, end, visited); length >= 0 {
				if maxLength < length+edge.length {
					maxLength = length + edge.length
				}
			}
		}
	}
	return maxLength
}

func solve(grid [][]byte) int {
	junctions := findJunctions(grid)
	graph := buildGraph(grid, junctions)
	start := Pos{0, 1}
	end := Pos{len(grid) - 1, len(grid[0]) - 2}
	return findLongestPath(graph, start, end, make(map[Pos]bool))
}

func main() {
	grid := readInput("input.txt")
	result := solve(grid)
	fmt.Println("Longest hike length without slope restrictions:", result)
}
```

## Links

[If you're new to Advent of Code, it's an annual event that takes place throughout December, featuring a series of programming puzzles that get progressively more challenging as Christmas approaches.](https://adventofcode.com/2023/day/23)

<details>
	<summary>Click to show the input</summary>
	<pre>
#.###########################################################################################################################################
#.........#...#.....###...#...###...#...#####...#...#...#...###.........#...###...#####...#...#...#...#...#...............#.......#####...###
#########.#.#.#.###.###.#.#.#.###.#.#.#.#####.#.#.#.#.#.#.#.###.#######.#.#.###.#.#####.#.#.#.#.#.#.#.#.#.#.#############.#.#####.#####.#.###
#.........#.#.#...#.#...#.#.#...#.#.#.#.#.....#.#.#...#.#.#...#...#.....#.#...#.#...#...#.#.#...#.#.#...#.#.........#.....#.....#.....#.#.###
#.#########.#.###.#.#.###.#.###.#.#.#.#.#.#####.#.#####.#.###.###.#.#####.###.#.###.#.###.#.#####.#.#####.#########.#.#########.#####.#.#.###
#.....#...#.#...#.#.#...#.#.#...#.#.#.#.#.....#.#.#.....#...#.#...#.#...#.#...#.#...#.#...#.#.....#.....#...###...#.#.......#...#.....#.#.###
#####.#.#.#.###.#.#.###.#.#.#.###.#.#.#.#####.#.#.#.#######.#.#.###.#.#.#.#.###.#.###.#.###.#.#########.###.###.#.#.#######.#.###.#####.#.###
#.....#.#.#.#...#.#.#...#.#.#...#.#.#.#.#.....#.#.#.......#.#.#...#.#.#.#.#.###.#.###.#.#...#...#...###...#.....#...#.......#...#.#.....#...#
#.#####.#.#.#v###.#.#.###.#.###.#.#.#.#.#.#####.#.#######.#.#.###.#.#.#.#.#.###.#.###.#.#.#####.#.#.#####.###########.#########.#.#.#######.#
#.....#.#...#.>.#.#.#...#.#...#.#.#.#.#.#...#...#.#.......#.#.#...#.#.#.#.#...#.#.>.>.#...#.....#.#.#...#.........#...#...#.....#.#...#.....#
#####.#.#####v#.#.#.###.#.###.#.#.#.#.#.###.#.###.#.#######.#.#.###.#.#.#.###.#.###v#######.#####.#.#.#.#########.#.###.#.#.#####.###.#.#####
#.....#.#.....#.#.#.#...#...#.#...#.#.#.>.>.#.#...#.#.>.>...#.#...#.#.#.#.###...#...#.....#.#...#.#.#.#.#.......#.#.#...#.#.....#.....#.....#
#.#####.#.#####.#.#.#.#####.#.#####.#.###v###.#.###.#.#v#####.###.#.#.#.#.#######.###.###.#.#.#.#.#.#.#.#.#####.#.#.#.###.#####.###########.#
#.....#.#.....#.#.#.#...#...#.#.....#.#...#...#...#...#...#...#...#...#.#.#.....#.#...###...#.#.#.#.#.#...#.....#.#.#.#...#...#...#.........#
#####.#.#####.#.#.#.###.#.###.#.#####.#.###.#####.#######.#.###.#######.#.#.###.#.#.#########.#.#.#.#.#####.#####.#.#.#.###.#.###.#.#########
#.....#.#.....#...#.#...#.#...#.......#...#.....#.#.......#...#...#...#...#...#...#.....#...#.#.#.#.#.#.....#.....#.#.#...#.#...#.#.#.......#
#.#####.#.#########.#.###.#.#############.#####.#.#.#########.###.#.#.#######.#########.#.#.#.#.#.#.#.#.#####.#####.#.###.#.###.#.#.#.#####.#
#.......#.........#...###...#.......#.....#...#...#.........#.....#.#.#.....#.......#...#.#.#.#.#.#.#.#.#...#.....#.#.#...#...#.#.#.#.#.....#
#################.###########.#####.#.#####.#.#############.#######.#.#.###.#######.#.###.#.#.#.#.#.#.#.#.#.#####.#.#.#.#####.#.#.#.#.#.#####
#...###...#.......###...#...#.....#.#.#.....#...#...........###...#.#.#...#.........#...#.#...#.#.#...#.#.#.>.>...#.#.#.#...#.#...#...#...###
#.#.###.#.#.#########.#.#.#.#####.#.#.#.#######.#.#############.#.#.#.###.#############.#.#####.#.#####.#.###v#####.#.#.#.#.#.###########.###
#.#.....#.#.....#...#.#.#.#.....#.#.#...#...#...#.....#...#...#.#.#.#...#.#.......#.....#...#...#.#...#.#.###.....#.#.#...#...###...#...#...#
#.#######.#####.#.#.#.#.#.#####.#.#.#####.#.#.#######.#.#.#.#.#.#.#.###.#.#.#####.#.#######.#.###.#.#.#.#.#######.#.#.###########.#.#.#.###.#
#.......#.#.....#.#.#.#.#.#...#.#.#...#...#...#...###...#.#.#...#.#.#...#...#.....#.......#.#.#...#.#.#...#####...#...###...#...#.#.#.#.#...#
#######.#.#.#####.#.#.#.#.#.#.#.#.###.#.#######.#.#######.#.#####.#.#.#######.###########.#.#.#.###.#.#########.#########.#.#.#.#.#.#.#.#.###
#.......#.#.....#.#.#.#.#...#.#.#.#...#...#.....#.###...#.#.....#.#.#.......#.......#...#...#...###.#...#...#...#...###...#...#...#.#.#.#...#
#.#######.#####.#.#.#.#.#####.#.#.#.#####.#.#####.###.#.#.#####.#.#.#######.#######.#.#.###########.###.#.#.#.###.#.###.###########.#.#.###.#
#...#...#.......#.#.#.#.#...#.#...#...#...#.....#.....#...#.....#.#.#.......###.....#.#.###...#...#.#...#.#.#...#.#...#...........#.#.#...#.#
###.#.#.#########.#.#.#.#.#.#v#######.#.#######.###########.#####.#.#.#########v#####.#.###.#.#.#.#.#.###.#.###v#.###.###########.#.#.###.#.#
#...#.#...#.......#...#.#.#.>.>.#...#...###.....#.........#.....#.#.#.#...#...>.>.#...#...#.#.#.#...#...#.#.#.>.>.#...#...........#...#...#.#
#.###.###.#.###########.#.###v#.#.#.#######.#####.#######.#####.#.#.#.#.#.#.###v#.#.#####.#.#.#.#######.#.#.#.#v###.###.###############.###.#
#.....#...#...........#.#.#...#...#.......#.....#.#.......###...#.#.#.#.#...#...#...###...#.#.#.......#.#.#.#.#...#...#.....#.........#...#.#
#######.#############.#.#.#.#############.#####.#.#.#########.###.#.#.#.#####.#########.###.#.#######.#.#.#.#.###.###.#####.#.#######.###.#.#
#.......#...#...#.....#...#...#.....#.....#...#...#.....###...#...#.#...#...#.........#...#.#.#.......#...#...#...###...###...#.......###.#.#
#.#######.#.#.#.#.###########.#.###.#.#####.#.#########.###.###.###.#####.#.#########.###.#.#.#.###############.#######.#######.#########.#.#
#.......#.#.#.#.#.......#####.#...#...#.....#.#...###...#...###...#.#.....#.......#...###.#.#...###...#.......#.....###...#.....#...#...#...#
#######.#.#.#.#.#######.#####.###.#####.#####.#.#.###.###.#######.#.#.###########.#.#####.#.#######.#.#.#####.#####.#####.#v#####.#.#.#.#####
#.......#.#.#.#.#...#...#.....#...#...#.....#.#.#.#...###.....###...#...........#.#.#...#.#.#.......#.#.....#.#...#.....#.>.#...#.#...#.#...#
#.#######.#.#.#.#.#.#.###.#####.###.#.#####.#.#.#.#.#########.#################.#.#.#.#.#.#.#.#######.#####.#.#.#.#####.###v#.#.#.#####.#.#.#
#...#...#.#.#.#...#.#...#.....#.###.#.#...#.#.#.#.#...###...#.....#...........#.#.#...#.#...#.......#.......#...#.#...#.###...#...#...#.#.#.#
###.#.#v#.#.#.#####.###.#####.#.###.#.#.#.#.#.#.#.###v###.#.#####.#.#########.#.#.#####.###########.#############.#.#.#.###########.#.#.#.#.#
###...#.>.#...#...#.....#...#.#...#.#.#.#.#.#.#.#.#.>.>...#.......#.....#...#...#.#...#.....#.......#...........#...#...#...........#...#.#.#
#######v#######.#.#######.#.#.###.#.#.#.#.#.#.#.#.#.#v#################.#.#.#####.#.#.#####.#.#######.#########.#########.###############.#.#
#.....#.........#.#...#...#...###...#...#.#.#.#.#...#.#...###.........#.#.#.....#.#.#.#.....#...#...#.#.........###.......#...#...###...#.#.#
#.###.###########.#.#.#.#################.#.#.#.#####.#.#.###.#######.#.#.#####.#.#.#.#.#######.#.#.#.#.###########.#######.#.#.#.###.#.#.#.#
#...#.#...........#.#.#.#...#...#...#...#...#.#.....#...#...#...#.....#...#.....#...#...###...#...#...#...........#...#...#.#.#.#...#.#.#.#.#
###.#.#.###########.#.#.#.#.#.#.#.#.#.#.#####.#####.#######.###.#.#########.###############.#.###################.###.#.#.#.#.#.###.#.#.#.#.#
#...#.#.............#.#.#.#.#.#.#.#.#.#.#...#...#...###.....#...#...#...###.................#.###...#...#.........#...#.#.#.#...###...#...#.#
#.###.###############.#.#.#.#.#.#.#.#.#.#.#.###.#.#####.#####.#####.#.#.#####################.###.#.#.#.#.#########.###.#.#.###############.#
#...#.............#...#...#...#...#...#.#.#...#...#...#.#...#.....#...#...#.......#.....#...#.#...#...#.#.........#...#.#.#.#.............#.#
###.#############.#.###################.#.###.#####.#.#.#.#.#####.#######.#.#####.#.###.#.#.#.#.#######.#########.###.#.#.#.#.###########.#.#
###.#.....###...#...#...###...###...###...#...#...#.#.#.#.#.#.....#.......#.....#.#...#...#...#.......#...........###...#...#.#...#.....#.#.#
###.#.###.###.#.#####.#.###.#.###.#.#######v###.#.#.#.#.#.#.#.#####.###########.#.###.###############.#######################.#.#.#.###.#.#.#
#...#.#...#...#...#...#.#...#.....#.......>.>...#.#.#.#...#...#.....###...#.....#.....###...#...#.....#...#.......#...#...###...#...#...#...#
#.###.#.###.#####.#.###.#.#################v#####.#.#.#########.#######.#.#.#############.#.#.#.#.#####.#.#.#####.#.#.#.#.###########.#######
#...#.#...#...#...#.###.#...#...............#.....#.#.#...#...#.....###.#.#.....###...###.#.#.#.#.......#...#.....#.#.#.#...#...#...#.......#
###.#.###v###.#.###.###.###.#.###############.#####.#.#.#.#.#.#####v###.#.#####v###.#.###.#.#.#.#############.#####.#.#.###.#.#.#.#.#######.#
###...###.>...#.#...#...#...#.#...#...#.....#.....#.#.#.#...#.....>.>.#.#...#.>.>...#...#.#...#.#...#.........#.....#...#...#.#...#.###.....#
#########v#####.#.###.###.###.#.#.#.#.#.###.#####.#.#.#.###########v#.#.###.#.#v#######.#.#####.#.#.#.#########.#########.###.#####.###.#####
###...###.#...#...###...#.###.#.#...#...#...#.....#.#.#.#.......###.#.#.###...#.#.....#.#.#...#.#.#.#.........#.....#.....###.....#...#.....#
###.#.###.#.#.#########.#.###.#.#########.###.#####.#.#.#.#####.###.#.#.#######.#.###.#.#.#.#.#.#.#.#########.#####.#.###########.###.#####.#
#...#.....#.#.###...#...#...#...#.........###.....#.#...#.....#.#...#.#.#.......#.#...#...#.#...#.#.#...#.....#.....#.#...#...###...#.....#.#
#.#########.#.###.#.#.#####.#####.###############.#.#########.#.#.###.#.#.#######.#.#######.#####.#.#.#.#.#####.#####.#.#.#.#.#####.#####.#.#
#.#...#.....#...#.#.#...#...#...#.............###...#.......#.#.#...#.#.#...#...#.#.#...###.....#.#...#.#...###.#.....#.#.#.#.#...#.....#...#
#.#.#.#.#######.#.#.###.#.###.#.#############.#######.#####.#.#.###.#.#.###.#.#.#.#.#.#.#######.#.#####.###.###.#.#####.#.#.#.#.#.#####v#####
#...#...#.......#.#.###...###.#.#.............#.....#.#...#...#.....#...###.#.#.#.#...#.....#...#...#...#...#...#...#...#.#.#.#.#...#.>.#...#
#########.#######.#.#########.#.#.#############.###.#.#.#.#################.#.#.#.#########.#.#####.#.###v###.#####.#.###.#.#.#.###.#.#v#.#.#
#.........#.....#.#.....###...#.#...............#...#...#.#...............#.#.#.#.#...#.....#...#...#.#.>.>...#####...#...#.#.#...#...#...#.#
#.#########.###.#.#####.###.###.#################.#######.#.#############.#.#.#.#.#.#.#.#######.#.###.#.#v#############.###.#.###.#########.#
#...#...#...###...#.....#...#...#.................#.....#...###.......#...#.#.#...#.#...#######.#.###.#.#.......###...#.#...#.....###.......#
###.#.#.#.#########.#####.###.###.#################.###.#######.#####.#.###.#.#####.###########.#.###.#.#######.###.#.#.#.###########.#######
#...#.#.#.#.........###...#...###.........#...#...#...#.........#...#...###...#.....#.........#.#...#...#...#...#...#.#...#.....#...#.......#
#.###.#.#.#.###########.###.#############.#.#.#.#.###.###########.#.###########.#####.#######.#.###.#####.#.#.###.###.#####.###.#.#.#######.#
#.....#...#...........#...#.....#...#.....#.#.#.#...#.......#.....#...#...#...#.......#.......#.#...###...#...###...#.#...#...#.#.#...#.....#
#####################.###.#####.#.#.#.#####.#.#.###.#######.#.#######.#.#.#.#.#########.#######.#.#####.###########.#.#.#.###v#.#.###.#.#####
#.......###...........###.#.....#.#.#.....#.#.#.#...#.....#...#.....#.#.#...#...........###...#...#...#.........###.#.#.#.#.>.#...###.#...###
#.#####.###.#############.#.#####.#.#####v#.#.#.#.###.###.#####.###.#.#.###################.#.#####.#.#########v###.#.#.#.#.#v#######.###.###
#.#...#.....#.....#...#...#.#.....#...#.>.>.#.#.#...#.#...#.....###...#.......#.............#.#.....#.........>.>.#.#.#.#.#.#...#...#...#...#
#.#.#.#######.###.#.#.#.###.#.#######.#.#v###.#.###.#.#.###.#################.#.#############.#.###############v#.#.#.#.#.#.###.#.#.###.###.#
#.#.#...#...#.#...#.#...#...#.....###...#...#...#...#.#...#.......#...#...###...###...........#.......#.......#.#.#.#...#.#.###...#.###...#.#
#.#.###.#.#.#.#.###.#####.#######.#########.#####.###.###.#######.#.#.#.#.#########.#################.#.#####.#.#.#.#####.#.#######.#####.#.#
#...#...#.#.#.#...#.#...#...#...#.#.....#...#.....###.#...#.......#.#.#.#.....#...#.........#...#.....#.....#...#...###...#...#.....#...#...#
#####.###.#.#.###.#.#.#.###.#.#.#.#.###.#.###.#######.#.###.#######.#.#.#####.#.#.#########.#.#.#.#########.###########.#####.#.#####.#.#####
#...#...#.#...#...#...#...#...#...#...#.#...#.......#.#...#.......#.#.#.#.....#.#.#...#.....#.#.#.......###.....###.....#.....#.#.....#.....#
#.#.###.#.#####.#########.###########.#.###.#######.#.###.#######.#.#.#.#.#####.#.#.#.#v#####.#.#######.#######.###.#####.#####.#.#########.#
#.#...#...#.....#####...#...###...#...#...#.#.......#.#...#...#...#.#.#.#.#...#.#.#.#.>.>.#...#...#...#.#.....#...#.....#.#####.#.#.........#
#.###.#####.#########.#.###.###.#.#.#####.#.#.#######.#.###.#.#v###.#.#.#.#.#.#.#.#.###v#.#.#####.#.#.#.#.###.###.#####.#.#####.#.#.#########
#...#.#.....#...###...#.....#...#...#...#.#.#.....#...#.....#.>.>.#.#.#.#.#.#...#...###.#.#.#.....#.#...#...#.....#.....#...#...#.#...#...###
###.#.#.#####.#v###.#########.#######.#.#.#.#####.#.###########v#.#.#.#.#.#.###########.#.#.#.#####.#######.#######.#######.#.###.###.#.#.###
#...#...#.....#.>.#...#...###.....###.#.#...#.....#...#.......#.#...#...#...#.....#.....#...#.....#...#...#.......#.#...#...#...#.###...#...#
#.#######.#####v#.###.#.#.#######.###.#.#####.#######.#.#####.#.#############.###.#.#############.###.#.#.#######.#.#.#.#.#####.#.#########.#
#.........#...#.#.#...#.#.....###.....#.....#.#...#...#.....#.#...........###...#.#...........#...#...#.#.........#.#.#...#####...#...#.....#
###########.#.#.#.#.###.#####.#############.#.#.#.#.#######.#.###########.#####.#.###########.#.###.###.###########.#.#############.#.#.#####
#...###.....#...#...###.....#.###...........#...#...###.....#.............#...#.#.....#.......#.....###...........#...###...#...#...#...#####
#.#.###.###################.#.###.#####################.###################.#.#.#####.#.#########################.#######.#.#.#.#.###########
#.#...#...................#.#...#.......#...#.....#...#...............#.....#...#.....#...........#...............###...#.#.#.#...#.........#
#.###.###################.#.###.#######.#.#.#.###.#.#.###############.#.#########.###############.#.#################.#.#.#.#.#####.#######.#
#...#.#...###.........#...#...#...#.....#.#.#.#...#.#.#...#...#.....#.#.........#.....#.....#.....#...........#...#...#...#.#.......###...#.#
###.#.#.#.###.#######.#.#####.###.#.#####.#.#.#.###.#.#.#.#.#.#.###.#.#########.#####.#.###.#.###############.#.#.#.#######.###########.#.#.#
#...#...#.....#.....#...###...###...#.....#...#.#...#...#...#.#...#...#...#####.....#.#...#.#.....###.........#.#...#.......#...........#...#
#.#############.###.#######.#########.#########.#.###########.###.#####.#.#########.#.###.#.#####.###.#########.#####.#######.###############
#.....#.....#...#...#.....#.....#.....#...#.....#.#.....#...#.....###...#.#.........#.....#...#...#...#.........#...#...#.....#.............#
#####.#.###.#.###v###.###.#####.#.#####.#.#.#####.#.###.#.#.#########.###.#.#################.#.###.###.#########.#.###.#.#####.###########.#
#...#...###...###.>.#...#.###...#.#...#.#.#.#...#...###...#.........#...#.#.........#.....###...###.....###.....#.#.....#.......###.......#.#
#.#.#############v#.###.#.###v###.#.#.#.#.#.#.#.###################.###.#.#########.#.###.#################.###.#.#################.#####.#.#
#.#.#...#.........#...#.#.#.>.>...#.#...#...#.#.#.......#...........#...#...#.......#.#...#...#...#...#.....###...#...#.....#...#...#...#...#
#.#.#.#.#.###########.#.#.#.#v#####.#########.#.#.#####.#.###########.#####.#.#######.#.###.#.#.#.#.#.#.###########.#.#.###.#.#.#.###.#.#####
#.#.#.#.#.......#####.#.#.#.#.#...#.#...#...#.#.#.....#.#...........#.....#.#.......#.#...#.#...#...#.#...........#.#.#...#.#.#.#...#.#.....#
#.#.#.#.#######.#####.#.#.#.#.#.#.#.#.#.#.#.#.#.#####.#.###########.#####.#.#######.#.###.#.#########.###########.#.#.###.#.#.#.###.#.#####.#
#.#...#.#...###.....#.#.#.#.#.#.#...#.#.#.#...#.#...#.#.#...........#...#.#.###.....#.#...#...#.......#...#.......#.#...#.#.#.#...#...#...#.#
#.#####.#.#.#######.#.#.#.#.#.#.#####.#.#.#####.#.#.#.#.#.###########.#.#.#.###v#####.#.#####.#.#######.#.#v#######.###.#.#.#.###.#####.#.#.#
#.....#.#.#.#.....#.#...#...#.#.#...#.#.#.#.....#.#.#.#.#...###.....#.#...#.#.>.>...#.#.#...#.#...#.....#.>.>...###.#...#.#.#.#...#...#.#.#.#
#####.#.#.#.#.###.#.#########.#.#.#.#.#.#.#.#####.#.#.#.###v###.###.#.#####.#.#v###.#.#.#.#.#.###.#.#######v###.###.#.###.#.#.#.###.#.#v#.#.#
#.....#...#.#.###.#...###.....#.#.#.#.#.#.#.#...#.#.#.#...>.>...###.#.....#.#.#...#...#.#.#.#.###...###...#.###...#.#.#...#.#.#.#...#.>.#...#
#.#########.#.###.###.###.#####.#.#.#.#.#.#.#.#.#.#.#.#####v#######.#####.#.#.###.#####.#.#.#.#########.#.#.#####.#.#.#.###.#.#.#.#####v#####
#.#...#...#.#...#...#.#...#...#...#...#...#.#.#.#.#.#.#.....#...###...#...#.#...#.....#.#.#.#.#.....#...#...#.....#.#.#.###.#.#...#...#.#...#
#.#.#.#.#.#.###.###.#.#.###.#.#############.#.#.#.#.#.#.#####.#.#####.#.###.###.#####.#.#.#.#.#.###.#.#######.#####.#.#.###.#.#####.#.#.#.#.#
#...#...#.#.#...###...#...#.#...#...#...###...#.#.#.#.#.#.....#.....#.#.###.....#.....#...#...#...#.#.......#.......#.#.###...#...#.#.#...#.#
#########.#.#.###########.#.###.#.#.#.#.#######.#.#.#.#.#.#########.#.#.#########.###############.#.#######.#########.#.#######.#.#.#.#####.#
#.........#...###...###...#.###...#...#.......#.#.#...#...#.....#...#.#.###...###.............###.#.........###.....#...#...###.#...#.......#
#.###############.#.###.###.#################.#.#.#########.###.#.###.#.###.#.###############.###.#############.###.#####.#.###.#############
#...............#.#...#.....#.................#...###.......###.#...#.#.#...#.#...............#...#...#...###...#...###...#...#.............#
###############.#.###.#######.#######################.#########.###.#.#.#.###.#.###############.###.#.#.#.###.###.#####.#####.#############.#
###.............#.#...###...#.......#...#...#.......#.........#.#...#...#...#.#.............###.#...#.#.#...#.#...#...#...#...#.......#.....#
###.#############.#.#####.#.#######.#.#.#.#.#.#####.#########.#.#.#########.#.#############.###.#.###v#.###.#.#.###.#.###.#.###.#####.#.#####
#...#.....#...###.#.#...#.#.#...###.#.#.#.#...#.....#.........#...###...#...#.............#...#...#.>.>.#...#.#...#.#.#...#...#.....#.#.....#
#.###.###.#.#.###.#.#.#.#.#.#.#.###v#.#.#.#####.#####.###############.#.#.###############.###.#####.#####.###.###.#.#.#.#####.#####.#.#####.#
#.#...#...#.#.#...#.#.#.#.#.#.#.#.>.>.#...#.....#...#...........###...#.#...#...........#.....###...#.....###...#.#.#.#...#...#...#.#.#...#.#
#.#.###.###.#.#.###.#.#.#.#.#.#.#.#########.#####.#.###########.###.###.###.#.#########.#########.###.#########.#.#.#.###.#.###.#.#.#.#v#.#.#
#.#.#...#...#.#...#.#.#.#.#.#.#.#.#.........#...#.#...#.........#...#...###...#.........###...#...###...#.....#.#.#.#...#.#...#.#.#.#.>.#...#
#.#.#.###.###.###.#.#.#.#.#.#.#.#.#.#########.#.#.###.#.#########.###.#########.###########.#.#.#######.#.###.#.#.#.###.#.###.#.#.#.###v#####
#.#.#...#.###...#.#.#.#.#.#...#...#.........#.#.#.#...#.........#...#.#...#...#.........#...#.#.....#...#.###.#.#.#.#...#.###...#.#.###.....#
#.#.###.#.#####.#.#.#.#.#.#################.#.#.#.#.###########.###.#.#.#.#.#.#########.#.###.#####.#.###.###.#.#.#.#.###.#######.#.#######.#
#...###.#.#.....#.#...#.#.......#...........#.#.#.#...#...#.....###.#.#.#.#.#.#.....#...#...#.#.....#...#.###.#.#...#...#.#.......#...#.....#
#######.#.#.#####.#####.#######.#.###########.#.#.###.#.#.#.#######.#.#.#.#.#.#.###.#.#####.#.#.#######.#.###.#.#######.#.#.#########.#.#####
#.......#.#.#...#.....#...#.....#.......#...#.#.#...#.#.#.#.#...###.#.#.#.#.#.#...#.#.#...#.#.#.....#...#.#...#...#.....#.#...#...#...#.#...#
#.#######.#.#.#.#####.###.#.###########.#.#.#.#.###.#.#.#.#v#.#.###.#.#.#.#.#.###.#.#v#.#.#.#.#####.#.###.#.#####.#.#####.###.#.#.#.###.#.#.#
#.#...#...#.#.#.#.....###.#.#...........#.#.#.#...#.#.#.#.>.>.#.#...#.#.#.#.#.#...#.>.>.#.#.#.#...#.#...#.#...#...#.#...#.#...#.#.#.#...#.#.#
#.#.#.#.###.#.#.#.#######.#.#.###########.#.#.###.#.#.#.#######.#.###.#.#.#.#.#.#########.#.#.#.#.#.###.#.###.#.###.#.#.#.#.###.#.#.#.###.#.#
#...#...###...#...#######...#.............#...###...#...#######...###...#...#...#########...#...#...###...###...###...#...#.....#...#.....#.#
###########################################################################################################################################.#
  </pre>
</details>