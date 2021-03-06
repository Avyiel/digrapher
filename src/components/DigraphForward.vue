<template>
	<v-card class="elevation-12">
		<v-toolbar color="primary" dark flat>
			<v-toolbar-title>Knoten + Adjazenzliste der Endknoten</v-toolbar-title>
			<v-spacer></v-spacer>
			<v-tooltip bottom>
				<template v-slot:activator="{ on }">
					<v-btn icon large v-on="on" @click="onReset()">
						<v-icon>mdi-restore</v-icon>
					</v-btn>
				</template>
				<span>Reset</span>
			</v-tooltip>
		</v-toolbar>
		<v-card-text>
			<v-form>
				<v-text-field
					v-model="nodesInput"
					label="Knoten"
					name="nodes"
					type="text"
					outlined
				></v-text-field>
				<v-text-field
					v-model="edgesInput"
					label="Adjazenzliste"
					name="edges"
					type="text"
					outlined
				></v-text-field>
			</v-form>

			<v-row dense>
				<v-col class="text-right" cols="4">
					<v-label>Pfeile</v-label>
				</v-col>
				<v-col>{{ print_l(edges) }}</v-col>
			</v-row>

			<v-row dense>
				<v-col class="text-right" cols="4">
					<v-label>Inzidenzliste</v-label>
				</v-col>
				<v-col>{{ incidenceList }}</v-col>
			</v-row>

			<v-row dense>
				<v-col class="text-right" cols="4">
					<v-label>Adjazenzliste der Startknoten</v-label>
				</v-col>
				<v-col>{{ sourceAdjacencyList }}</v-col>
			</v-row>

			<v-row dense>
				<v-col class="text-right" cols="4">
					<v-label>Forward Star EK</v-label>
				</v-col>
				<v-col>{{ ek }}</v-col>
			</v-row>

			<v-row dense>
				<v-col class="text-right" cols="4">
					<v-label>Forward Start EKI</v-label>
				</v-col>
				<v-col>{{ eki }}</v-col>
			</v-row>
		</v-card-text>
		<v-card-actions>
			<v-spacer></v-spacer>
			<v-btn color="primary" @click="calculate()">Calculate</v-btn>
		</v-card-actions>
	</v-card>
</template>

<script lang="ts">
import { difference, isEqual, union } from 'lodash'
import { Component, Prop, Vue } from 'vue-property-decorator'
import { INode } from '../models/node.model'
import { print_r } from '../utils/print'

@Component
export default class DigraphForward extends Vue {
	private nodesInput = ''
	private edgesInput = ''

	private graph: Array<INode> = []

	constructor() {
		super()
	}

	get nodes(): Array<number> {
		return this.nodesInput
			.trim()
			.slice(1, this.nodesInput.length - 1)
			.split(',')
			.map(x => Number(x))
	}

	get edges(): Array<Array<number>> {
		const output: Array<Array<number>> = []
		const parse = this.edgesInput
			.trim()
			.slice(2, this.edgesInput.length - 2)
			.split('],[')
			.map(y =>
				y.split(',{').map(z =>
					z
						.replace('}', '')
						.split(',')
						.map(k => Number(k))
				)
			)

		for (let index = 0; index < parse.length; index++) {
			const el = parse[index]
			if (el[0][0] > 0) {
				el[1].forEach(x => {
					output.push([index + 1, x])
				})
			}
		}

		return output
	}

	get incidenceList(): string {
		function _formatArray(obj: Array<any>) {
			if (obj.length === 0) return '[]'

			let retVal = '['
			obj.forEach(x => (retVal += `${x},`))
			retVal = retVal.slice(0, retVal.length - 1)
			retVal += ']'

			return retVal
		}

		let output = '['

		this.graph.forEach(node => {
			output += '['
			output += node.degreeOut
			output += ',{'
			const edges: Array<Array<number>> = []
			this.edges.forEach(edge => {
				if (
					edge[0] - 1 == this.graph.indexOf(node) &&
					edges.indexOf(edge) === -1
				) {
					output += _formatArray(edge)
					output += ','
					edges.push(edge)
				}
			})
			if (output.charAt(output.length - 1) !== '{')
				output = output.slice(0, output.length - 1)

			output += '}'
			output += '],'
		})

		if (output.charAt(output.length - 1) !== '[')
			output = output.slice(0, output.length - 1)
		output += ']'
		return output
	}

	get sourceAdjacencyList(): string {
		let output = '['

		this.graph.forEach(node => {
			output += '['
			output += node.degreeIn
			output += ','
			output += print_r(node.prev)
			output += '],'
		})

		if (output.charAt(output.length - 1) !== '[')
			output = output.slice(0, output.length - 1)

		output += ']'
		return output
	}

	get ek(): Array<number> {
		const output = [1]
		let index = 0
		this.graph.forEach(node => {
			output.push(node.degreeOut + output[index++])
		})

		return output
	}

	get eki(): Array<number> {
		const output: Array<number> = []
		this.graph.forEach(node => {
			node.next.forEach(next => output.push(next))
		})

		return output
	}

	private calculate(): void {
		this.graph = []

		this.nodes.forEach(node =>
			this.graph.push({
				next: [],
				prev: [],
				degreeOut: 0,
				degreeIn: 0
			})
		)

		this.edges.forEach(edge => {
			this.graph[edge[0] - 1].next.push(edge[1])
			this.graph[edge[1] - 1].prev.push(edge[0])
			this.graph[edge[0] - 1].degreeOut += 1
			this.graph[edge[1] - 1].degreeIn += 1
		})
	}

	private onReset() {
		this.nodesInput = ''
		this.edgesInput = ''
		this.graph = []
	}
}
</script>

<style scoped></style>
