<template>
  <div class="container">
    <div class="controls">
      <button
        class="actionButtons"
        @click="
          () => {
            presentAlgorithm()
            message = ''
          }
        "
      >
        present Algorithm
      </button>
    </div>

    <div class="grid">
      <div class="row" v-for="row in grid" :key="JSON.stringify(row)">
        <div
          class="point"
          v-for="e in row"
          :key="JSON.stringify(e)"
          :style="{ background: getPointColor(e), borderRadius: e.border }"
          @click="
            () => {
              grid[e.row][e.col].isWall = !grid[e.row][e.col].isWall
              calculateGridPointsRadius(e)
            }
          "
        />
      </div>

      <div v-if="!!message" @click="message = ''" class="message">
        <div class="message-text">
          {{ message.split('\n')[0] }}
          <br />
          <br />
          {{ message.split('\n')[1] }}
        </div>
      </div>
    </div>

    <div class="controls">
      <button
        class="actionButtons"
        @click="
          () => {
            startAlgorithm()
            message = ''
          }
        "
      >
        start
      </button>
      <button
        class="actionButtons"
        @click="
          () => {
            resetGrid()
            message = ''
          }
        "
      >
        reset Grid
      </button>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

type point = {
  col: number
  row: number
  f: number
  h: number
  g: number
  border: string
  isWall: boolean
  previous: undefined | { col: number; row: number }
  neighbors: Array<{ col: number; row: number }>
}

const border = 50
const useDiagonals = true
const fullness = 0.65
const rows = 35
const columns = 35

export default Vue.extend({
  data() {
    const grid: Array<Array<point>> = [...Array(rows)].map((e1, row) =>
      [...Array(columns)].map((e2, col) => {
        const neighbors = []

        const isWall = false

        if (col + 1 < columns) neighbors.push({ col: col + 1, row: row })
        if (col - 1 >= 0) neighbors.push({ col: col - 1, row: row })
        if (row + 1 < rows) neighbors.push({ col: col, row: row + 1 })
        if (row - 1 >= 0) neighbors.push({ col: col, row: row - 1 })

        if (useDiagonals) {
          if (col + 1 < columns && row + 1 < rows)
            neighbors.push({ col: col + 1, row: row + 1 })
          if (col - 1 >= 0 && row - 1 >= 0)
            neighbors.push({ col: col - 1, row: row - 1 })
          if (row + 1 < rows && col - 1 >= 0)
            neighbors.push({ col: col - 1, row: row + 1 })
          if (row - 1 >= 0 && col + 1 < columns)
            neighbors.push({ col: col + 1, row: row - 1 })
        }

        return {
          col,
          row,
          neighbors,
          isWall,
          f: 0,
          g: 0,
          h: 0,
          border: '',
          previous: undefined,
        }
      })
    )

    const start: point = grid[0][0]
    const end: point = grid[rows - 1][columns - 1]

    end.isWall = false
    start.isWall = false

    return {
      columns,
      rows,
      grid,
      start,
      end,
      path: [] as Array<{ row: number; col: number }>,
      message: '',
      openSet: [start] as Array<point>,
      closedSet: [] as Array<point>,
    }
  },

  mounted() {
    this.calculateGridPointsRadius()
  },

  methods: {
    startAlgorithm() {
      this.calculateGridPointsRadius()
      this.calculateAlgorithm()
    },

    presentAlgorithm() {
      const grid: Array<Array<point>> = [...Array(rows)].map((e1, row) =>
        [...Array(columns)].map((e2, col) => {
          const neighbors = []

          const isWall = Math.random() > fullness

          if (col + 1 < columns) neighbors.push({ col: col + 1, row: row })
          if (col - 1 >= 0) neighbors.push({ col: col - 1, row: row })
          if (row + 1 < rows) neighbors.push({ col: col, row: row + 1 })
          if (row - 1 >= 0) neighbors.push({ col: col, row: row - 1 })

          if (useDiagonals) {
            if (col + 1 < columns && row + 1 < rows)
              neighbors.push({ col: col + 1, row: row + 1 })
            if (col - 1 >= 0 && row - 1 >= 0)
              neighbors.push({ col: col - 1, row: row - 1 })
            if (row + 1 < rows && col - 1 >= 0)
              neighbors.push({ col: col - 1, row: row + 1 })
            if (row - 1 >= 0 && col + 1 < columns)
              neighbors.push({ col: col + 1, row: row - 1 })
          }

          return {
            col,
            row,
            neighbors,
            isWall,
            f: 0,
            g: 0,
            h: 0,
            border: '',
          } as point
        })
      )

      const start: point = grid[0][0]
      const end: point = grid[rows - 1][columns - 1]

      end.isWall = false
      start.isWall = false

      this.columns = columns
      this.rows = rows
      this.grid = grid
      this.start = start
      this.end = end
      this.path = []
      this.openSet = [start] as Array<point>
      this.closedSet = [] as Array<point>

      this.calculateGridPointsRadius()
      this.calculateAlgorithm()
    },

    resetGrid() {
      const grid: Array<Array<point>> = [...Array(rows)].map((e1, row) =>
        [...Array(columns)].map((e2, col) => {
          const neighbors = []

          const isWall = false

          if (col + 1 < columns) neighbors.push({ col: col + 1, row: row })
          if (col - 1 >= 0) neighbors.push({ col: col - 1, row: row })
          if (row + 1 < rows) neighbors.push({ col: col, row: row + 1 })
          if (row - 1 >= 0) neighbors.push({ col: col, row: row - 1 })

          if (useDiagonals) {
            if (col + 1 < columns && row + 1 < rows)
              neighbors.push({ col: col + 1, row: row + 1 })
            if (col - 1 >= 0 && row - 1 >= 0)
              neighbors.push({ col: col - 1, row: row - 1 })
            if (row + 1 < rows && col - 1 >= 0)
              neighbors.push({ col: col - 1, row: row + 1 })
            if (row - 1 >= 0 && col + 1 < columns)
              neighbors.push({ col: col + 1, row: row - 1 })
          }

          return {
            col,
            row,
            neighbors,
            isWall,
            f: 0,
            g: 0,
            h: 0,
            border: '',
            previous: undefined,
          }
        })
      )

      const start: point = grid[0][0]
      const end: point = grid[rows - 1][columns - 1]

      end.isWall = false
      start.isWall = false

      this.columns = columns
      this.rows = rows
      this.grid = grid
      this.start = start
      this.end = end
      this.path = []
      this.openSet = [start] as Array<point>
      this.closedSet = [] as Array<point>

      this.calculateGridPointsRadius()
    },

    calculateGridPointsRadius() {
      this.grid.forEach((i) =>
        i.forEach(
          (j) => (this.grid[j.row][j.col].border = this.getPointBorder(j))
        )
      )
    },

    calculateAlgorithm() {
      if (this.openSet.length > 0) {
        let winner = 0
        for (let i = 0; i < this.openSet.length; i++) {
          if (this.openSet[i].f < this.openSet[winner].f) winner = i
        }

        const current = this.openSet[winner]

        if (current.col === this.end.col && current.row === this.end.row) {
          let tmp = this.grid[current.row][current.col]
          let pathTmp: Array<{
            row: number
            col: number
          }> = [{ col: tmp.col, row: tmp.row }]

          while (!!tmp.previous) {
            pathTmp.push({ col: tmp.previous.col, row: tmp.previous.row })
            tmp = this.grid[tmp.previous.row][tmp.previous.col]
          }

          this.path = pathTmp

          this.message = 'PATH FOUND ! \n ┬─┬ノ( º _ ºノ)'
          return
        }

        this.removeFromArray(this.openSet, current)
        this.closedSet.push(current)

        const neighbors = this.grid[current.row][current.col].neighbors
        for (let i = 0; i < neighbors.length; i++) {
          const neighbor = this.grid[neighbors[i].row][neighbors[i].col]

          if (
            !this.closedSet.some(
              (e) => e.row === neighbor.row && e.col === neighbor.col
            ) &&
            !neighbor.isWall
          ) {
            const tempG = current.g + 1

            if (
              this.openSet.some(
                (e) => e.col === neighbor.col && e.row === neighbor.row
              )
            ) {
              if (tempG < neighbor.g) {
                neighbor.g = tempG
              }
            } else {
              neighbor.g = tempG
              this.openSet.push(neighbor)
            }

            neighbor.h = this.calculateDistance(neighbor, this.end)
            neighbor.f = neighbor.g + neighbor.h
            neighbor.previous = { col: current.col, row: current.row }
          }
        }
      } else {
        this.message = 'NO PATH FOUND \n (╯°□°)╯︵ ┻━┻'
        return
      }

      setTimeout(() => this.calculateAlgorithm())
    },

    calculateDistance(p1: point, p2: point) {
      return Math.abs(p1.col - p2.col) + Math.abs(p1.row - p2.row)
    },

    removeFromArray(arr: Array<point>, point: point) {
      for (let i = arr.length - 1; i >= 0; i--) {
        if (arr[i].col == point.col && arr[i].row === point.row) {
          arr.splice(i, 1)
        }
      }
    },

    getPointColor(point: point) {
      if (this.path.some((e) => point.col === e.col && point.row === e.row))
        return 'deepskyblue'
      if (point.col === this.start.col && point.row === this.start.row)
        return 'black'
      if (point.col === this.end.col && point.row === this.end.row)
        return 'black'
      if (point.isWall) return 'transparent'
      if (
        this.closedSet.some((e) => e.col === point.col && e.row === point.row)
      )
        return 'yellowgreen'
      if (this.openSet.some((e) => e.col === point.col && e.row === point.row))
        return 'green'
      return '#d4fc79'
    },

    getPointBorder(point: point) {
      let topRadius = 1
      if (
        point.neighbors.some(
          (e) =>
            e.row < point.row &&
            e.col === point.col &&
            !!this.grid[e.row][e.col] &&
            !this.grid[e.row][e.col].isWall
        )
      )
        topRadius = 0

      let bottomRadius = 1
      if (
        point.neighbors.some(
          (e) =>
            e.row > point.row &&
            e.col === point.col &&
            !!this.grid[e.row][e.col] &&
            !this.grid[e.row][e.col].isWall
        )
      )
        bottomRadius = 0

      let leftRadius = 1
      if (
        point.neighbors.some(
          (e) =>
            e.col < point.col &&
            e.row === point.row &&
            !!this.grid[e.row][e.col] &&
            !this.grid[e.row][e.col].isWall
        )
      )
        leftRadius = 0

      let rightRadius = 1
      if (
        point.neighbors.some(
          (e) =>
            e.col > point.col &&
            e.row === point.row &&
            !!this.grid[e.row][e.col] &&
            !this.grid[e.row][e.col].isWall
        )
      )
        rightRadius = 0

      return `${leftRadius + topRadius === 2 ? border : 0}px ${
        rightRadius + topRadius === 2 ? border : 0
      }px ${rightRadius + bottomRadius === 2 ? border : 0}px ${
        leftRadius + bottomRadius === 2 ? border : 0
      }px`
    },
  },
})
</script>

<style>
.container {
  display: grid;
  justify-content: center;
  align-content: center;
}

.message {
  position: absolute;
  width: 100%;
  height: 100%;
  display: grid;
  align-content: center;
  font-weight: bold;
  color: rgb(240, 226, 147);
  border-radius: 28px;
  background: rgba(0, 0, 0, 0.7);
  font-weight: bold;
  text-transform: uppercase;
  font-size: 48px;
  letter-spacing: 3px;
}

.message:hover {
  cursor: pointer;
}

.message-text {
  padding: 30px;
  background-color: rgba(0, 0, 0, 0.7);
  text-align: center;
  width: 100%;
}

.controls {
  display: grid;
  grid-gap: 10px;
  justify-content: center;
  align-content: center;
  padding: 10px;
}

.grid {
  position: relative;
  margin: auto;
  padding: 20px;
  box-shadow: inset 0 0 10px #000000;
  border-radius: 30px;
  background-color: rgb(53, 50, 50);
  display: grid;
  justify-content: center;
  align-content: center;
  grid-gap: 2px;
}

.row {
  display: grid;
  grid-gap: 2px;
  grid-template-columns: repeat(35, auto);
}

.point {
  width: 20px;
  height: 20px;
  background: yellowgreen;
  transition: 0.3s;

  display: grid;
  justify-content: center;
  align-content: center;
  font-weight: bold;
  font-family: monospace;
}

.actionButtons {
  width: 300px;
  outline: none;
  font-family: sans-serif;
  padding: 10px;
  font-size: 14px;
  border: none;
  border-radius: 10px;
  font-weight: bold;
  text-transform: uppercase;
  background: rgb(53, 50, 50);
  color: #d4fc79;
  letter-spacing: 3px;
  box-shadow: inset 0 0 10px #000000;
}

.actionButtons:hover {
  cursor: pointer;
}
</style>
