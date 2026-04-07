<script setup>
import { computed, ref } from 'vue'
import lightIcon from '../assets/aspect/光.png'
import iceIcon from '../assets/aspect/冰.png'
import earthIcon from '../assets/aspect/地.png'
import phantomIcon from '../assets/aspect/幻.png'
import ghostIcon from '../assets/aspect/幽.png'
import evilIcon from '../assets/aspect/恶.png'
import normalIcon from '../assets/aspect/普.png'
import mechIcon from '../assets/aspect/机.png'
import martialIcon from '../assets/aspect/武.png'
import poisonIcon from '../assets/aspect/毒.png'
import waterIcon from '../assets/aspect/水.png'
import fireIcon from '../assets/aspect/火.png'
import electricIcon from '../assets/aspect/电.png'
import wingIcon from '../assets/aspect/翼.png'
import grassIcon from '../assets/aspect/草.png'
import cuteIcon from '../assets/aspect/萌.png'
import bugIcon from '../assets/aspect/虫.png'
import dragonIcon from '../assets/aspect/龙.png'

const types = [
  { key: 'normal', name: '普系', iconSrc: normalIcon, color: '#478bb6' },
  { key: 'grass', name: '草系', iconSrc: grassIcon, color: '#58b978' },
  { key: 'fire', name: '火系', iconSrc: fireIcon, color: '#df5529' },
  { key: 'water', name: '水系', iconSrc: waterIcon, color: '#6699e0' },
  { key: 'light', name: '光系', iconSrc: lightIcon, color: '#53ade0' },
  { key: 'earth', name: '地系', iconSrc: earthIcon, color: '#9e8342' },
  { key: 'ice', name: '冰系', iconSrc: iceIcon, color: '#63a9d2' },
  { key: 'dragon', name: '龙系', iconSrc: dragonIcon, color: '#ec4b67' },
  { key: 'electric', name: '电系', iconSrc: electricIcon, color: '#e5c400' },
  { key: 'poison', name: '毒系', iconSrc: poisonIcon, color: '#ab5ad0' },
  { key: 'bug', name: '虫系', iconSrc: bugIcon, color: '#9ecf1f' },
  { key: 'martial', name: '武系', iconSrc: martialIcon, color: '#fa973b' },
  { key: 'wing', name: '翼系', iconSrc: wingIcon, color: '#4abdc0' },
  { key: 'cute', name: '萌系', iconSrc: cuteIcon, color: '#e976aa' },
  { key: 'ghost', name: '幽系', iconSrc: ghostIcon, color: '#8e4bdf' },
  { key: 'evil', name: '恶系', iconSrc: evilIcon, color: '#cc4b84' },
  { key: 'mech', name: '机系', iconSrc: mechIcon, color: '#4ec8aa' },
  { key: 'phantom', name: '幻系', iconSrc: phantomIcon, color: '#969ce7' },
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

const selectedDefenders = ref([])

const selectedDefenderA = computed(() => selectedDefenders.value[0] ?? null)
const selectedDefenderB = computed(() => selectedDefenders.value[1] ?? null)
const selectedDefenderTypes = computed(() => {
  const selected = selectedDefenders.value
    .map((key) => types.find((type) => type.key === key))
    .filter(Boolean)

  while (selected.length < 2) {
    selected.push(null)
  }

  return selected
})

const toggleDefender = (typeKey) => {
  const current = selectedDefenders.value
  const existingIndex = current.indexOf(typeKey)

  if (existingIndex !== -1) {
    selectedDefenders.value = current.filter((key) => key !== typeKey)
    return
  }

  if (current.length < 2) {
    selectedDefenders.value = [...current, typeKey]
    return
  }

  selectedDefenders.value = [current[1], typeKey]
}

const resetSelectedDefenders = () => {
  selectedDefenders.value = []
}

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
    const hasDualSelection = selectedDefenders.value.length === 2
    const dualValue = hasDualSelection
      ? getEffectValue(attacker.key, selectedDefenderA.value) * getEffectValue(attacker.key, selectedDefenderB.value)
      : null

    return {
      attacker,
      dualCell: {
        value: dualValue,
        state: dualValue === null ? 'normal' : getStateByMultiplier(dualValue),
        label: dualValue === null ? '-' : formatMultiplier(dualValue),
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
      <button
        type="button"
        class="reset-btn"
        :disabled="selectedDefenders.length === 0"
        @click="resetSelectedDefenders"
      >
        重置
      </button>
    </header>

    <div class="matrix-shell">
      <div class="matrix-canvas">
        <table class="matrix-table">
          <thead>
            <tr>
              <th class="corner">攻击方</th>
              <th class="dual-head">
                <div class="dual-title">
                  <span class="dual-icons">
                    <template v-for="(type, index) in selectedDefenderTypes" :key="`picked-${index}`">
                      <img
                        v-if="type"
                        class="dual-picked-icon"
                        :src="type.iconSrc"
                        :alt="type.name"
                      />
                      <i v-else class="dual-empty-dot" aria-hidden="true"></i>
                    </template>
                  </span>
                  <small>{{ selectedDefenders.length }}/2</small>
                </div>
              </th>
              <th v-for="type in types" :key="`head-${type.key}`">
                <button
                  type="button"
                  class="type-pill-btn"
                  :class="{ selected: selectedDefenders.includes(type.key) }"
                  :style="{ '--defense-color': type.color }"
                  @click="toggleDefender(type.key)"
                >
                <span class="type-pill">
                  <img class="type-icon-img" :src="type.iconSrc" :alt="type.name" />
                  {{ type.name }}
                </span>
                </button>
              </th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="row in matrix"
              :key="row.attacker.key"
              :class="{
                recommended:
                  selectedDefenders.length === 2 && selectedDefenders.includes(row.attacker.key),
              }"
            >
              <th class="row-head">
                <button
                  type="button"
                  class="attack-btn"
                  :style="{ '--attack-color': row.attacker.color }"
                >
                  <span class="icon">
                    <img class="attack-icon-img" :src="row.attacker.iconSrc" :alt="row.attacker.name" />
                  </span>
                  <span class="label">{{ row.attacker.name }}</span>
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
  height: 100vh;
  display: flex;
  flex-direction: column;
  padding: 0.45rem 0.6rem 0.45rem;
  background:
    radial-gradient(circle at 12% 20%, rgba(242, 198, 114, 0.22), transparent 26%),
    radial-gradient(circle at 88% 78%, rgba(79, 156, 170, 0.25), transparent 32%),
    #ece7db;
  color: #3f3a34;
  overflow: hidden;
}

.title-wrap {
  text-align: center;
  margin-bottom: 0.28rem;
}

.reset-btn {
  margin-top: 0.28rem;
  height: 28px;
  min-width: 72px;
  border: 0;
  border-radius: 8px;
  background: #2f2f34;
  color: #f6f3ea;
  font-size: 0.72rem;
  font-weight: 700;
  cursor: pointer;
  transition: opacity 0.2s ease, transform 0.2s ease;
}

.reset-btn:hover:not(:disabled) {
  transform: translateY(-1px);
}

.reset-btn:disabled {
  opacity: 0.45;
  cursor: not-allowed;
}

h1 {
  margin: 0;
  font-size: clamp(0.95rem, 2vw, 1.3rem);
  letter-spacing: 0.04em;
}

.matrix-shell {
  width: 100%;
  flex: 1;
  min-height: 0;
  margin: 0;
  border: 0;
  border-radius: 0;
  background: transparent;
  box-shadow: none;
  display: flex;
  justify-content: center;
  padding: 0;
  overflow: auto;
}

.matrix-canvas {
  width: max-content;
  min-width: max-content;
  margin: 0 auto;
}

.matrix-table {
  width: max-content;
  border-collapse: separate;
  border-spacing: 8px 5px;
  padding: 8px 12px;
}

th,
td {
  width: 52px;
  height: 33px;
  background: #f2eee4;
  border: 1px solid #cdc5b6;
  border-radius: 11px;
  text-align: center;
  overflow: hidden;
}

thead th {
  height: 44px;
}

thead th:not(.corner):not(.dual-head) {
  border-color: transparent;
  background: transparent;
  padding: 0;
}

.corner {
  width: 92px;
  min-width: 92px;
  font-size: 0.94rem;
  color: #4f4b45;
  background: #e7e0d2;
}

.dual-head {
  width: 96px;
  min-width: 96px;
  background: #e9e3d6;
}

.dual-title {
  display: grid;
  gap: 2px;
  place-items: center;
  font-size: 0.76rem;
  color: #4a443c;
}

.dual-icons {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
}

.dual-picked-icon {
  width: 15px;
  height: 15px;
  object-fit: cover;
  border-radius: 2px;
}

.dual-empty-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #b6ae9e;
  display: inline-block;
}

.dual-title small {
  font-size: 0.68rem;
  color: #6d665a;
}

.type-pill-btn {
  width: 100%;
  height: 100%;
  border: 1px solid #d4cbbb;
  border-radius: 10px;
  background: #f7f3ea;
  color: var(--defense-color);
  cursor: pointer;
  box-shadow: inset 3px 0 0 var(--defense-color);
  transition: transform 0.2s ease, box-shadow 0.2s ease, background-color 0.2s, color 0.2s ease;
}

.type-pill-btn.selected {
  background: #2f2f34;
  box-shadow: inset 0 0 0 2px #2f2f34;
}

.type-pill-btn:hover {
  transform: translateY(-1px);
  box-shadow: inset 3px 0 0 var(--defense-color), 0 6px 16px rgba(70, 60, 46, 0.18);
}

.type-pill {
  display: inline-grid;
  place-items: center;
  gap: 1px;
  font-size: 0.92rem;
  font-weight: 700;
  line-height: 1.2;
  color: var(--defense-color);
}

.type-pill-btn.selected .type-pill {
  color: #f6f3ea;
}

.type-icon-img {
  width: 16px;
  height: 16px;
  object-fit: cover;
  border-radius: 2px;
}

.row-head {
  width: 92px;
  min-width: 92px;
  background: transparent;
  border-color: transparent;
  padding: 0;
}

.dual-col {
  width: 96px;
  min-width: 96px;
}

.attack-btn {
  width: 100%;
  height: 100%;
  border: 1px solid #c9bfad;
  border-radius: 10px;
  background: #f8f4eb;
  color: var(--attack-color);
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.34rem;
  font-size: 0.92rem;
  font-weight: 700;
  cursor: pointer;
  box-shadow: inset 3px 0 0 var(--attack-color);
  transition: transform 0.2s ease, box-shadow 0.2s ease, background-color 0.2s, color 0.2s ease;
}

.attack-btn:hover {
  transform: translateY(-1px);
  box-shadow: inset 3px 0 0 var(--attack-color), 0 6px 16px rgba(70, 60, 46, 0.18);
}

.attack-btn .icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.attack-icon-img {
  width: 16px;
  height: 16px;
  object-fit: cover;
  border-radius: 2px;
}

.attack-btn .label {
  color: var(--attack-color);
}

.attack-btn.picked {
  background: #2f2f34;
  color: #f6f3ea;
}

.attack-btn.picked .label {
  color: #f6f3ea;
}

tbody tr.active td {
  background: #ebe4d5;
}

tbody tr.recommended td {
  background: #d5efc8;
  border-color: #9fca87;
}

tbody tr.recommended .dual-col {
  background: #c3e8af;
  border-color: #88bb6d;
}

tbody tr.recommended .attack-btn {
  box-shadow: inset 5px 0 0 #74b55a;
}

.result {
  width: 27px;
  height: 21px;
  border-radius: 8px;
  border: 1px solid rgba(55, 48, 40, 0.22);
  display: inline-grid;
  place-items: center;
  font-size: 0.96rem;
  font-weight: 800;
  color: #fff;
}

.dual-col .result {
  width: 58px;
  height: 21px;
  font-size: 0.84rem;
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

  .matrix-table {
    border-spacing: 6px 4px;
    padding: 4px 6px;
  }

  th,
  td {
    width: 46px;
    height: 29px;
  }

  .corner,
  .row-head {
    width: 82px;
    min-width: 82px;
  }

  .dual-head,
  .dual-col {
    width: 80px;
    min-width: 80px;
  }

  .dual-col .result {
    width: 50px;
    height: 20px;
    font-size: 0.76rem;
  }

  .dual-picked-icon {
    width: 13px;
    height: 13px;
  }

  .attack-btn {
    font-size: 0.82rem;
  }

  .type-pill {
    font-size: 0.82rem;
  }

  .type-icon-img,
  .attack-icon-img {
    width: 14px;
    height: 14px;
  }
}
</style>
