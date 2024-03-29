<template>
  <HeadlineActions>
    <template #headline>
      <div class="flex flex-col">
        <div>
          {{ assetSymbol.name }}
        </div>
        <div class="text-base text-gray-500 font-medium">
          {{ assetSymbol.tickerName }}
        </div>
      </div>
    </template>
    <template #default>
      <BuySellAsset :asset-symbol="assetSymbol" />
      <WatchAssetModal :asset-symbol="assetSymbol" />
    </template>
  </HeadlineActions>
  <div class="p-6 mb-6 bg-white rounded-lg">
    <Suspense>
      <AssetPrice :symbol="assetSymbol" />
    </Suspense>
  </div>
  <div class="rounded-lg bg-white overflow-hidden p-6 mb-6">
    <Suspense>
      <CandlestickChart :asset-symbol="assetSymbol" />
      <template #fallback>
        <GSkeleton style="height: 500px" />
      </template>
    </Suspense>
  </div>
  <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
    <div>
      <CompanyQuote :asset-symbol="assetSymbol" />
    </div>
    <div>
      <CompanyInfo :asset-symbol="assetSymbol" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { AssetSymbol } from '@common/models';
import BuySellAsset from '@components/TransactionModal.vue';
import CandlestickChart from '@components/Charts/CandleStickChart/CandlestickChart.vue';
import CompanyInfo from '@components/CompanyInfo.vue';
import CompanyQuote from '@components/CompanyQuote.vue';
import HeadlineActions from '@components/HeadlineActions.vue';
import WatchAssetModal from '@components/WatchAssetModal.vue';
import { onUnmounted, ref } from 'vue';
import AssetPrice from '@components/AssetPrice.vue';
import GSkeleton from '@components/base/GSkeleton.vue';

const props = defineProps<{
  ticker: string;
}>();

const liveSubscription = ref<Parse.LiveQuerySubscription>();
const assetSymbol = ref<AssetSymbol>();

try {
  assetSymbol.value = await AssetSymbol.fetchSymbolByTicker(props.ticker);
} catch (e) {
  alert(e);
}

if (liveSubscription.value) {
  await liveSubscription.value.unsubscribe();
}

onUnmounted(async () => {
  if (liveSubscription.value) {
    await liveSubscription.value.unsubscribe();
  }
});
</script>
