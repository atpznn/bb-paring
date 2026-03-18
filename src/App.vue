<template>
  <div class="min-h-screen bg-gradient-to-br from-purple-50 to-pink-50">
    <!-- Header -->
    <header class="bg-white shadow-sm border-b border-purple-100">
      <div class="max-w-md mx-auto px-4 py-4">
        <h1 class="text-2xl font-bold text-center text-purple-800 text-shadow">
          จับคู่แบทมิน 🔥
        </h1>
      </div>
    </header>

    <!-- Main Content -->
    <main class="max-w-md mx-auto px-4 py-6">
      <!-- Input Section -->
      <section class="bg-white rounded-2xl shadow-lg p-6 mb-6">
        <div class="space-y-4">
          <div>
            <label
              for="userInput"
              class="block text-sm font-medium text-gray-700 mb-2"
            >
              รายชื่อผู้เล่น
            </label>
            <textarea
              id="userInput"
              v-model="userInput"
              placeholder="ใส่รายชื่อผู้เล่น คั่นด้วยคอมม่าหรือบรรทัดใหม่&#10;เช่น: ผู้เล่น1, ผู้เล่น2, ผู้เล่น3"
              class="w-full px-4 py-3 border border-gray-300 rounded-xl focus:ring-2 focus:ring-purple-500 focus:border-transparent resize-none transition-all"
              rows="4"
            ></textarea>
          </div>

          <div>
            <label
              for="lockPairs"
              class="block text-sm font-medium text-gray-700 mb-2"
            >
              ล็อคคู่
              <div>(เเยกคู่ด้วย "," ขึ้นบรรทัดใหม่เพื่อเพิ่มคู่ใหม่)</div>
            </label>
            <textarea
              id="lockPairs"
              v-model="lockPairs"
              placeholder="คู่ที่ต้องการล็อค เช่น: ผู้เล่น1 ผู้เล่น2, ผู้เล่น3 ผู้เล่น4"
              class="w-full px-4 py-3 border border-gray-300 rounded-xl focus:ring-2 focus:ring-purple-500 focus:border-transparent resize-none transition-all"
              rows="2"
            ></textarea>
          </div>

          <button
            @click="generatePairs"
            :disabled="!userInput.trim() || isGenerating"
            class="w-full bg-gradient-to-r from-purple-600 to-pink-600 text-white font-semibold py-3 px-6 rounded-xl hover:from-purple-700 hover:to-pink-700 disabled:opacity-50 disabled:cursor-not-allowed transition-all transform active:scale-95"
          >
            <span v-if="isGenerating">กำลังจับคู่...</span>
            <span v-else>จับคู่! 🎲</span>
          </button>
        </div>
      </section>

      <!-- Results Section -->
      <section v-if="pairingResults.length > 0" class="space-y-4">
        <div class="flex justify-between items-center">
          <h2 class="text-lg font-semibold text-gray-800">
            ผลลัพธ์การจับคู่ ({{ pairingResults.length }} รอบ)
          </h2>
          <button
            @click="showRandomResult"
            class="bg-green-500 text-white px-4 py-2 rounded-lg hover:bg-green-600 transition-colors text-sm font-medium"
          >
            สุ่มเล่น 🎯
          </button>
        </div>

        <!-- Results Cards -->
        <div class="space-y-3 max-h-96 overflow-y-auto">
          <div
            v-for="(result, index) in pairingResults"
            :key="index"
            class="bg-white rounded-xl shadow-md p-4 border border-purple-100 hover:shadow-lg transition-shadow"
          >
            <div class="flex justify-between items-center mb-3">
              <span class="text-sm font-medium text-purple-600"
                >รอบที่ {{ index + 1 }}</span
              >
              <button
                @click="selectResult(index)"
                class="text-xs bg-purple-100 text-purple-700 px-2 py-1 rounded-full hover:bg-purple-200 transition-colors"
              >
                เลือกรอบนี้
              </button>
            </div>

            <div class="flex flex-col gap-2">
              <div
                v-for="(pair, pairIndex) in result"
                :key="pairIndex"
                class="bg-gradient-to-r from-purple-50 to-pink-50 rounded-lg p-2 text-center"
              >
                <span class="text-sm font-medium text-gray-800">
                  {{ pair[0] }} คู่กับ {{ pair[1] }}
                </span>
              </div>
            </div>
          </div>
        </div>
      </section>
    </main>

    <!-- Random Result Dialog -->
    <div
      v-if="showDialog"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center p-4 z-50"
      @click="closeDialog"
    >
      <div
        class="bg-white rounded-2xl shadow-2xl max-w-sm w-full p-6 transform transition-all"
        @click.stop
      >
        <div class="text-center">
          <h3 class="text-xl font-bold text-purple-800 mb-4">
            🎉 ผลลัพธ์สุ่ม!
          </h3>

          <div v-if="selectedResult" class="space-y-3">
            <div class="text-sm text-gray-600 mb-2">
              รอบที่ {{ selectedRound + 1 }}
            </div>

            <div class="grid grid-cols-2 gap-2">
              <div
                v-for="(pair, pairIndex) in selectedResult"
                :key="pairIndex"
                class="bg-gradient-to-r from-purple-100 to-pink-100 rounded-lg p-3"
              >
                <span class="font-medium text-gray-800">
                  {{ pair[0] }} คู่กับ {{ pair[1] }}
                </span>
              </div>
            </div>
          </div>

          <button
            @click="closeDialog"
            class="mt-6 w-full bg-gradient-to-r from-purple-600 to-pink-600 text-white font-semibold py-2 px-4 rounded-lg hover:from-purple-700 hover:to-pink-700 transition-all"
          >
            ปิด
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";

// Types
type Pair = [string, string];
type Round = Pair[];
type PairingResults = Round[][][];

// Reactive state
const userInput = ref<string>("");
const lockPairs = ref<string>("");
const pairingResults = ref<PairingResults>([]);
const isGenerating = ref<boolean>(false);
const showDialog = ref<boolean>(false);
const selectedResult = ref<Round | null>(null);
const selectedRound = ref<number>(0);

function rotateCircular<T>(data: T[]): T[] {
  if (data.length <= 2) return [];
  const [first, second, ...rest] = data;
  return [first, ...rest, second];
}

interface ReponseSelective<T> {
  pair: T[];
  rest: T[];
}

function normalSelectivePair<T>(data: T[]): ReponseSelective<T> {
  const [first, ...rest] = data;
  const last = rest[rest.length - 1];
  return {
    pair: [first, last],
    rest: rest.slice(0, rest.length - 1),
  };
}

function reverseSelectivePair<T>(data: T[]): ReponseSelective<T> {
  const [first, second, ...rest] = data;
  return {
    pair: [first, second],
    rest: rest,
  };
}

function selectTivePair<T>(data: T[]): T[][] {
  const { pair, rest } = normalSelectivePair(data);
  if (rest.length == 0) return [pair];
  return [pair, ...selectTivePair(rest)];
}
function selectTivePairV2<T>(
  selective: (d: T[]) => ReponseSelective<T>,
  data: T[],
): T[][] {
  const { pair, rest } = selective(data);
  if (rest.length == 0) return [pair];
  return [pair, ...selectTivePair(rest)];
}
interface Holder<T> {
  pairs: T[][];
  rest: T[];
}
function selectTivePairV3<T>(
  selective: (d: T[]) => ReponseSelective<T>,
  data: T[],
): T[][] {
  return data.reduce<Holder<T>>(
    (state: Holder<T>, _) => {
      if (state.rest.length == 0) return state;
      const { pair, rest } = selective(state.rest);
      return { pairs: [...state.pairs, pair], rest } as Holder<T>;
    },
    { pairs: [], rest: data } as Holder<T>,
  ).pairs;
}

function createArray(length: number) {
  return Array.from({ length }, (_, i) => i + 1);
}

function isSameArray<T>(a: T[], b: T[]) {
  if (a.length != b.length) return false;
  return a.every((x, i) => x === b[i]);
}
function mapAllPosibleRotate<T>(
  rotation: (d: T[]) => T[],
  data: T[],
  visited: T[][] = [],
): T[][] {
  const rotated = rotation(data);
  if (visited.length == 0)
    return mapAllPosibleRotate(rotation, rotated, [data]);
  if (visited.some((s) => isSameArray(s, data))) return visited;
  return mapAllPosibleRotate(rotation, rotated, [...visited, data]);
}
function mapAllPosibleRotateV2<T>(rotation: (d: T[]) => T[], data: T[]): T[][] {
  return data.reduce((state, _) => {
    if (state.length == 0) return [data];
    const last = state[state.length - 1];
    const rotated = rotation(last);
    if (state.some((x) => isSameArray(rotated, x))) return state;
    return [...state, rotated];
  }, [] as T[][]);
}

function badmintonMakePair<T>(data: T[]): unknown[][][] {
  const makeAllPossible = mapAllPosibleRotateV2.bind(null, rotateCircular);
  const selectPairEachPossible = <T,>(d: T[][]) =>
    d.map(selectTivePairV3.bind(null, normalSelectivePair));
  return selectPairEachPossible(makeAllPossible(data));
}
function makeMatch<T>(maxnumber: number) {
  return function (data: T[][]) {
    return data.reduce((state: T[][][], pair: T[]) => {
      if (state.length == 0) {
        state.push([pair]);
        return state;
      }
      if (state[state.length - 1].length < maxnumber) {
        state[state.length - 1].push(pair);
        return state;
      }
      state.push([pair]);
      return state;
    }, []);
  };
}
function shuffle<T>(data: T[][]) {
  for (let i = data.length - 1; i > 0; i--) {
    let j = Math.floor(Math.random() * (i + 1));
    [data[i], data[j]] = [data[j], data[i]];
  }
  return data;
}
function display<T>(data: T[][][]) {
  data.forEach((s) => {
    console.log(s);
  });
}

// Empty function for you to implement
const generatePairingData = (userNames: string[]): any => {
  const lockPairsArray = lockPairs.value
    .split("\n")
    .map((line) => line.split(","))
    .map((names) => names.map((name) => name.trim()))
    .filter((names) => names.length > 0);

  // TODO: You can modify badmintonMakePair to accept lockPairsArray parameter
  const pair = badmintonMakePair(userNames);
  const shuffled = pair.map((x) => [...x, ...lockPairsArray]).map(shuffle);
  return shuffle(shuffled);
};

const generatePairs = async (): Promise<void> => {
  if (!userInput.value.trim()) return;

  isGenerating.value = true;

  // Parse user input
  const users = userInput.value
    .split(/[,\n]+/)
    .map((name) => name.trim())
    .filter((name) => name.length > 0);

  if (users.length < 2) {
    alert("กรุณาใส่ชื่อผู้เล่นอย่างน้อย 2 คน");
    isGenerating.value = false;
    return;
  }

  // Simulate processing time
  await new Promise((resolve) => setTimeout(resolve, 500));

  // Call the empty function - you'll implement this
  pairingResults.value = generatePairingData(users);

  isGenerating.value = false;
};

const showRandomResult = (): void => {
  if (pairingResults.value.length === 0) return;

  const randomIndex = Math.floor(Math.random() * pairingResults.value.length);
  selectResult(randomIndex);
};

const selectResult = (index: number): void => {
  selectedResult.value = pairingResults.value[index];
  selectedRound.value = index;
  showDialog.value = true;
};

const closeDialog = (): void => {
  showDialog.value = false;
  selectedResult.value = null;
  selectedRound.value = 0;
};
</script>
