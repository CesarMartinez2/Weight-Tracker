<script setup>
import { ref, shallowRef, onMounted, computed, watch, nextTick } from 'vue'
import Chart from 'chart.js/auto'
import { remove } from '@vue/shared';

const weights = ref([])

const weightChartEl = ref(null)

const weightChart = shallowRef(null)

const weightInput = ref(0)

const currentWeight = computed(() => {
	return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 }
})

const addWeight = () => {
	if (weightInput.value === '' || weightInput.value === 0 || weightInput.value < 0) {
		return
	} else {
		weights.value.push({
		weight: weightInput.value,
		date: new Date().getTime()
	})
	}

	weightInput.value = ''
}

const removeWeight = weight => {
	weights.value = weights.value.filter(t => t !== weight)
}

watch(weights, (newWeights) => {
	const ws = [...newWeights]

	if (weightChart.value) {
		weightChart.value.data.labels = ws
			.sort((a, b) => a.date - b.date)
			.map(weight => new Date(weight.date).toLocaleDateString() )
			.slice(-7)

		weightChart.value.data.datasets[0].data = ws
			.sort((a, b) => a.date - b.date)
			.map(weight => weight.weight)
			.slice(-7)

		weightChart.value.update()
		return
	}

	nextTick(() => {
		weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
			type: 'line',
			data: {
				labels: ws
					.sort((a, b) => a.date - b.date)
					.map(weight => new Date(weight.date).toLocaleDateString()),
				datasets: [
					{
						label: 'Weight (lbs)' ,
						data: ws
							.sort((a, b) => a.date - b.date)
							.map(weight => weight.weight),
						backgroundColor: 'rgba(0, 173, 212, 0.2)',
						borderColor: 'rgba(0, 173, 212, 1)',
						borderWidth: 1,
						fill: true
					}
				]
			},
			options: {
				responsive: true,
				maintainAspectRatio: false,
				plugins: {
					legend: {
							labels: {
								color: '#AAA'
							}
						}
				},
				scales: {
					x: {
						ticks: {
							color: '#AAA'
						},
					},
					y: {
						ticks: {
							color: '#AAA'
						},
					}
				}
			}
		})
	})
}, { deep: true })
</script>

<template>
	<main>

		<h1>Weight Tracker</h1>

		<div class="current">
			<span>{{ currentWeight.weight }}</span>
			<small>Current Weight (lbs)</small>
		</div>

		<form @submit.prevent="addWeight">
			<input 
				type="number"
				step="0.1"
				v-model="weightInput" />

			<input	
				type="submit"
				value="Add weight" />
		</form>

		<div v-if="weights && weights.length > 0">

			<h2>
				Last 7 Entries
			</h2>

			<div class="canvas-box">
				<canvas ref="weightChartEl"></canvas>
			</div>

			<div class="weight-history">
				<h2>Weight History</h2>
				<ul>
					<li v-for="weight in weights">
						<span>{{ weight.weight }}lbs</span>
						<small>
							{{ new Date(weight.date).toLocaleDateString() }}
						</small>
						<div class="actions">
							<button class="delete" @click="removeWeight(weight)">Delete</button>
						</div>
					</li>
				</ul>
			</div>

		</div>
	</main>
</template>
