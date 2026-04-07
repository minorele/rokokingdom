<script setup>
import { computed, ref } from 'vue'

const types = [
  { key: 'normal', name: '普通系', icon: '◎', color: '#7d8ca0' },
  { key: 'grass', name: '草系', icon: '❀', color: '#3ba668' },
  { key: 'fire', name: '火系', icon: '♨', color: '#df6f3d' },
  { key: 'water', name: '水系', icon: '◈', color: '#4a8cd9' },
  { key: 'light', name: '光系', icon: '✧', color: '#4db4d5' },
  { key: 'earth', name: '土系', icon: '▲', color: '#9a7c4d' },
  { key: 'ice', name: '冰系', icon: '❄', color: '#66a9ba' },
  { key: 'dragon', name: '龙系', icon: '✦', color: '#cb5663' },
  { key: 'electric', name: '电系', icon: '⚡', color: '#d4b036' },
  { key: 'poison', name: '毒系', icon: '☣', color: '#9b64d9' },
  { key: 'bug', name: '虫系', icon: '✿', color: '#8bb348' },
  { key: 'martial', name: '武系', icon: '✹', color: '#d09a4f' },
  { key: 'wing', name: '翼系', icon: '➹', color: '#3fa0b8' },
  { key: 'cute', name: '萌系', icon: '♡', color: '#d85f9f' },
  { key: 'ghost', name: '幽系', icon: '☯', color: '#7559b9' },
  { key: 'evil', name: '恶系', icon: '◍', color: '#b15575' },
  { key: 'mech', name: '机械系', icon: '⚙', color: '#5ca2a1' },
  { key: 'phantom', name: '幻系', icon: '◌', color: '#6a8bc9' },
]

const matchup = {
  normal: { strong: [], weak: ['earth', 'mech', 'ghost'] },
  grass: { strong: ['water', 'earth'], weak: ['fire', 'ice', 'bug', 'wing'] },
  fire: { strong: ['grass', 'ice', 'bug', 'mech'], weak: ['water', 'earth', 'dragon'] },
  water: { strong: ['fire', 'earth'], weak: ['grass', 'electric'] },
  light: { strong: ['ghost', 'evil'], weak: ['dragon', 'earth'] },
  earth: { strong: ['fire', 'electric', 'mech'], weak: ['grass', 'water', 'ice', 'wing'] },
  ice: { strong: ['grass', 'dragon', 'wing'], weak: ['fire', 'martial', 'mech'] },
  dragon: { strong: ['light', 'electric', 'fire'], weak: ['ice', 'cute'] },
  electric: { strong: ['water', 'wing'], weak: ['earth', 'dragon'] },
  poison: { strong: ['grass', 'cute'], weak: ['earth', 'ghost', 'mech'] },
  bug: { strong: ['grass', 'evil', 'light'], weak: ['fire', 'wing', 'martial'] },
  martial: { strong: ['normal', 'ice', 'evil', 'mech'], weak: ['light', 'ghost', 'cute'] },
  wing: { strong: ['grass', 'bug', 'martial'], weak: ['electric', 'ice'] },
  cute: { strong: ['dragon', 'martial', 'evil'], weak: ['poison', 'mech'] },
  ghost: { strong: ['ghost', 'poison'], weak: ['normal', 'light', 'evil']},
  evil: { strong: ['light', 'ghost'], weak: ['martial', 'cute', 'bug'] },
  mech: { strong: ['ice', 'cute', 'phantom'], weak: ['fire', 'earth', 'martial'] },
  phantom: { strong: ['light', 'dragon'], weak: ['mech', 'evil'] },
}

const activeAttack = ref(types[0].key)
const selectedDefenderA = ref(types[0].key)
const selectedDefenderB = ref(types[1].key)

const getEffectValue = (attackerKey, defenderKey) => {
  const info = matchup[attackerKey] ?? {}
  if (info.immune?.includes(defenderKey)) return 0
  if (info.strong?.includes(defenderKey)) return 2
  if (info.weak?.includes(defenderKey)) return 0.5
  return 1
}

const getStateByMultiplier = (value) => {
  if (value === 0) return 'immune'
  if (value === 4) return 'quad'
  if (value === 2) return 'strong'
  if (value === 1) return 'normal'
  if (value === 0.5) return 'weak'
  if (value === 0.25) return 'tiny'
  return 'normal'
}

const formatMultiplier = (value) => {
  if (Number.isInteger(value)) return `${value}x`
  return `${value}x`
}

const matrix = computed(() => {
  return types.map((attacker) => {
    const first = getEffectValue(attacker.key, selectedDefenderA.value)
    const second = getEffectValue(attacker.key, selectedDefenderB.value)
    const dualValue = first * second

    return {
      attacker,
      dualCell: {
        value: dualValue,
        state: getStateByMultiplier(dualValue),
        label: formatMultiplier(dualValue),
      },
      cells: types.map((defender) => {
        const value = getEffectValue(attacker.key, defender.key)
        return { value, state: getStateByMultiplier(value) }
      }),
    }
  })
})
</script>

<template>
  <section class="chart-page">
    <header class="title-wrap">
      <h1>洛克王国世界属性克制表</h1>
    </header>

    <div class="matrix-shell">
      <div class="matrix-scroll">
        <table class="matrix-table">
          <thead>
            <tr>
              <th class="corner">作为攻击方</th>
              <th class="dual-head">
                <div class="dual-picker">
                  <select v-model="selectedDefenderA" aria-label="第一个被攻击属性">
                    <option v-for="type in types" :key="`def-a-${type.key}`" :value="type.key">
                      {{ type.name }}
                    </option>
                  </select>
                  <select v-model="selectedDefenderB" aria-label="第二个被攻击属性">
                    <option v-for="type in types" :key="`def-b-${type.key}`" :value="type.key">
                      {{ type.name }}
                    </option>
                  </select>
                </div>
              </th>
              <th v-for="type in types" :key="`head-${type.key}`">
                <span class="type-pill" :style="{ '--pill-color': type.color }">
                  <i>{{ type.icon }}</i>
                  {{ type.name }}
                </span>
              </th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="row in matrix"
              :key="row.attacker.key"
              :class="{ active: activeAttack === row.attacker.key }"
            >
              <th class="row-head">
                <button
                  type="button"
                  class="attack-btn"
                  :class="{ picked: activeAttack === row.attacker.key }"
                  @click="activeAttack = row.attacker.key"
                >
                  <span class="icon">{{ row.attacker.icon }}</span>
                  <span>{{ row.attacker.name }}</span>
                </button>
              </th>
              <td class="dual-col">
                <span class="result" :class="row.dualCell.state">
                  {{ row.dualCell.label }}
                </span>
              </td>
              <td
                v-for="(cell, cellIndex) in row.cells"
                :key="`${row.attacker.key}-${types[cellIndex].key}`"
              >
                <span
                  v-if="cell.state !== 'normal'"
                  class="result"
                  :class="cell.state"
                >
                  <template v-if="cell.state === 'strong'">↑</template>
                  <template v-else-if="cell.state === 'weak'">↓</template>
                  <template v-else>×</template>
                </span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </section>
</template>

<style scoped>
.chart-page {
  min-height: 100vh;
  padding: 0.55rem 0.95rem 0.55rem;
  background:
    radial-gradient(circle at 12% 20%, rgba(242, 198, 114, 0.22), transparent 26%),
    radial-gradient(circle at 88% 78%, rgba(79, 156, 170, 0.25), transparent 32%),
    #ece7db;
  color: #3f3a34;
}

.title-wrap {
  text-align: center;
  margin-bottom: 0.28rem;
}

h1 {
  margin: 0;
  font-size: clamp(0.95rem, 2vw, 1.3rem);
  letter-spacing: 0.04em;
}

.matrix-shell {
  max-width: 1180px;
  margin: 0 auto;
  border: 3px solid #343238;
  border-radius: 14px;
  background: #f4f0e6;
  box-shadow: 0 14px 34px rgba(44, 37, 28, 0.18);
}

.matrix-scroll {
  overflow: auto;
  border-radius: 12px;
}

.matrix-table {
  width: max-content;
  border-collapse: separate;
  border-spacing: 6px 3px;
  padding: 5px 9px;
}

th,
td {
  width: 48px;
  height: 34px;
  background: #e8e3d7;
  border-radius: 10px;
  text-align: center;
}

thead th {
  height: 39px;
}

.corner {
  width: 132px;
  min-width: 132px;
  font-size: 0.8rem;
  color: #4f4b45;
  background: #ded8ca;
}

.dual-head {
  width: 86px;
  min-width: 86px;
  background: #e2ddcf;
}

.dual-picker {
  display: grid;
  gap: 2px;
  padding: 1px;
}

.dual-picker select {
  width: 100%;
  height: 16px;
  border: 0;
  border-radius: 5px;
  font-size: 0.6rem;
  background: #faf7ef;
  color: #4a443c;
}

.type-pill {
  display: inline-grid;
  place-items: center;
  gap: 1px;
  font-size: 0.64rem;
  line-height: 1.2;
  color: var(--pill-color);
}

.type-pill i {
  font-style: normal;
  font-size: 0.9rem;
}

.row-head {
  width: 132px;
  min-width: 132px;
  background: transparent;
}

.dual-col {
  width: 86px;
  min-width: 86px;
}

.attack-btn {
  width: 100%;
  height: 100%;
  border: 0;
  border-radius: 10px;
  background: #f5f2ea;
  color: #4f4841;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.28rem;
  font-size: 0.76rem;
  font-weight: 700;
  cursor: pointer;
  transition: transform 0.2s ease, box-shadow 0.2s ease, background-color 0.2s;
}

.attack-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 6px 16px rgba(70, 60, 46, 0.18);
}

.attack-btn .icon {
  font-size: 0.88rem;
}

.attack-btn.picked {
  background: #2f2f34;
  color: #f6f3ea;
}

tbody tr.active td {
  background: #f0eadf;
}

.result {
  width: 25px;
  height: 22px;
  border-radius: 8px;
  display: inline-grid;
  place-items: center;
  font-size: 0.82rem;
  font-weight: 800;
  color: #fff;
}

.dual-col .result {
  width: 54px;
  font-size: 0.72rem;
}

.result.normal {
  background: #c7c0b0;
  color: #3f3a33;
}

.result.strong {
  background: #62ad5f;
}

.result.weak {
  background: #d96a78;
}

.result.immune {
  background: #8f8b97;
}

.result.quad {
  background: #4f84e7;
}

.result.tiny {
  background: #8f5ed8;
}

@media (max-width: 820px) {
  .chart-page {
    padding: 0.6rem 0.45rem 0.55rem;
  }

  th,
  td {
    width: 44px;
    height: 30px;
  }

  .corner,
  .row-head {
    width: 110px;
    min-width: 110px;
  }

  .dual-head,
  .dual-col {
    width: 74px;
    min-width: 74px;
  }

  .dual-col .result {
    width: 46px;
    font-size: 0.66rem;
  }

  .dual-picker select {
    font-size: 0.56rem;
  }

  .attack-btn {
    font-size: 0.7rem;
  }
}
</style>
