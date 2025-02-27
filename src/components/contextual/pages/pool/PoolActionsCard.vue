<script setup lang="ts">
import { computed, toRef } from 'vue';
import useWithdrawMath from '@/components/forms/pool_actions/WithdrawForm/composables/useWithdrawMath';
import {
  isJoinsDisabled,
  usePoolHelpers,
  deprecatedDetails,
} from '@/composables/usePoolHelpers';
import useNetwork from '@/composables/useNetwork';
import { Pool } from '@/services/pool/types';
import useWeb3 from '@/services/web3/useWeb3';

import { Goals, trackGoal } from '@/composables/useFathom';
import { useDisabledJoinPool } from '@/composables/useDisabledJoinPool';

/**
 * TYPES
 */
type Props = {
  pool: Pool;
  missingPrices: boolean;
};

/**
 * PROPS
 */
const props = defineProps<Props>();

/**
 * COMPOSABLES
 */
const { hasBpt } = useWithdrawMath(toRef(props, 'pool'));
const { isMigratablePool, hasNonApprovedRateProviders } = usePoolHelpers(
  toRef(props, 'pool')
);
const { isWalletReady, startConnectWithInjectedProvider } = useWeb3();
const { networkSlug } = useNetwork();
const { shouldDisableJoins } = useDisabledJoinPool(props.pool);

/**
 * COMPUTED
 */
const joinDisabled = computed(
  (): boolean =>
    !!deprecatedDetails(props.pool.id) ||
    isJoinsDisabled(props.pool.id) ||
    hasNonApprovedRateProviders.value ||
    isMigratablePool(props.pool) ||
    shouldDisableJoins.value
);
</script>

<template>
  <div
    class="p-4 w-full bg-gray-50 dark:bg-gray-800 border-t border-gray-200 dark:border-gray-900"
  >
    <BalBtn
      v-if="!isWalletReady"
      :label="$t('connectWallet')"
      color="gradient"
      block
      @click="startConnectWithInjectedProvider"
    />
    <div v-else class="grid grid-cols-2 gap-2">
      <BalBtn
        :tag="joinDisabled ? 'div' : 'router-link'"
        :to="{ name: 'add-liquidity', params: { networkSlug } }"
        :label="$t('addLiquidity')"
        color="gradient"
        :disabled="joinDisabled"
        block
        @click="trackGoal(Goals.ClickAddLiquidity)"
      />

      <BalBtn
        :tag="hasBpt ? 'router-link' : 'div'"
        :to="{ name: 'withdraw', params: { networkSlug } }"
        :label="$t('withdraw.label')"
        :disabled="!hasBpt"
        color="blue"
        outline
        block
        @click="trackGoal(Goals.ClickWithdraw)"
      />
    </div>
  </div>
</template>
