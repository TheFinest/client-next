<template>
    <section class="page-discover-new">
        <ChartGrid>
            <template v-if="loading">
                <ChartItemPlaceholder
                    v-for="n in 10"
                    :key="n"
                />
            </template>
            <template v-else>
                <ChartItem
                    v-for="chart in charts"
                    :key="chart.id"
                    v-bind="chart"
                />
            </template>
        </ChartGrid>

        <div class="paginator">
            <button
                class="button"
                :disabled="currentPage === 0"
                @click="handlePrevious"
                v-interactable
            >
                <Remixicon icon="arrow-left" />
                <span>{{ $t('discover.previousPage') }}</span>
            </button>
            <button
                class="button"
                @click="handleNext"
                :disabled="!hasNextPage"
                v-interactable
            >
                <span>{{ $t('discover.nextPage') }}</span>
                <Remixicon icon="arrow-right" />
            </button>
        </div>
    </section>
</template>

<script setup>
import ChartGrid from '@/components/Charts/ChartGrid.vue';
import ChartItem from '@/components/Charts/ChartItem.vue';
import { inject, onMounted, ref, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import Remixicon from '@/components/Remixicon.vue';
import ChartItemPlaceholder from '@/components/Charts/ChartItemPlaceholder.vue';

const route = useRoute();
const router = useRouter();
const currentPage = ref(Number(route.params.page) || 0);
const charts = ref([]);
const loading = ref(true);
const hasNextPage = ref(false);
const api = inject('api');

async function loadPage(page) {
    hasNextPage.value = false;
    const apiCharts = await api.getHotThisWeekCharts(page);
    charts.value = apiCharts ?? [];
    loading.value = false;

    if (charts.value.length === 12) {
        const nextCharts = await api.getHotThisWeekCharts(page + 1);
        hasNextPage.value = (nextCharts?.length ?? 0) > 0;
    } else {
        hasNextPage.value = false;
    }
}

onMounted(() => loadPage(currentPage.value));

watch(() => route.params.page, (page) => {
    currentPage.value = Number(page) || 0;
    loadPage(currentPage.value);
});

function handlePrevious() {
    router.push({ name: route.name, params: { ...route.params, page: currentPage.value - 1 } });
}
function handleNext() {
    router.push({ name: route.name, params: { ...route.params, page: currentPage.value + 1 } });
}
</script>

<style scoped>
.page-discover-new {
    @apply p-10 flex flex-col gap-10;

    & .paginator {
        @apply flex gap-2 items-center justify-end;
    }
}
</style>
