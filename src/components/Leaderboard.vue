<template>
  <div class="max-w-3xl mx-auto py-8 px-4 relative" dir="rtl">

    <div class="text-center mb-10 flex flex-col items-center">
      <div
        class="w-28 h-28">
        <img src="/Logo.png" alt="شعار أياماً معدودات" class="w-full h-full object-cover">
      </div>

      <h1 class="text-4xl md:text-5xl font-bold text-ramadan-gold mb-2 tracking-wide">
        أياماً معدودات
      </h1>
      <p class="text-lg text-slate-400">لوحة صدارة المهام اليومية</p>
    </div>

    <div v-if="loading" class="flex justify-center items-center py-20">
      <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-ramadan-gold"></div>
    </div>

    <div v-else-if="error"
      class="bg-red-900/50 border border-red-500 text-red-200 p-4 rounded-lg text-center font-medium">
      <p>{{ error }}</p>
      <button @click="fetchLeaderboardData" class="mt-4 px-6 py-2 bg-red-800 hover:bg-red-700 rounded transition">
        حاول مرة أخرى
      </button>
    </div>

    <div v-else class="space-y-8 pb-16">

      <div
        class="flex justify-center items-center gap-3 bg-slate-800/60 p-4 rounded-xl border border-slate-700 w-full max-w-md mx-auto">
        <label for="dayFilter" class="text-slate-300 text-sm font-medium">عرض الترتيب لـ:</label>
        <select id="dayFilter" v-model="selectedDay"
          class="flex-1 bg-slate-900 text-ramadan-gold border border-slate-600 rounded-lg px-3 py-2 focus:outline-none focus:border-ramadan-gold transition-colors text-sm">
          <option value="all">المجموع الكلي (جميع الأيام)</option>
          <option v-for="day in availableDays" :key="day" :value="day">
            {{ day }}
          </option>
        </select>
      </div>

      <div v-if="topThree.length > 0" class="grid grid-cols-1 md:grid-cols-3 gap-6 pt-4">
        <div v-for="(user, index) in topThree" :key="user.alias"
          class="relative flex flex-col items-center p-6 rounded-xl border border-slate-700 bg-slate-800/50 shadow-lg backdrop-blur-sm"
          :class="{
            'border-yellow-400 shadow-yellow-900/20 md:-mt-6': index === 0,
            'border-slate-300 shadow-slate-700/20': index === 1,
            'border-amber-700 shadow-amber-900/20 md:mt-4': index === 2
          }">
          <div
            class="absolute -top-5 w-10 h-10 rounded-full flex items-center justify-center font-bold text-slate-900 text-lg shadow-md"
            :class="{
              'bg-yellow-400': index === 0,
              'bg-slate-300': index === 1,
              'bg-amber-600': index === 2
            }">
            {{ index + 1 }}
          </div>

          <h2 class="text-2xl font-bold mt-3 text-white tracking-widest flex items-center gap-2">
            {{ user.alias }}
            <span class="material-symbols-outlined text-lg">person</span>
          </h2>
          <p class="text-4xl font-bold mt-2" :class="{
            'text-yellow-400': index === 0,
            'text-slate-300': index === 1,
            'text-amber-600': index === 2
          }">
            {{ user.totalPoints }}
          </p>
          <span class="text-sm text-slate-400 mt-1">نقطة</span>
        </div>
      </div>

      <div v-if="sortedUsers.length === 0"
        class="text-center text-slate-400 py-10 bg-slate-800/30 rounded-xl border border-slate-700">
        لا توجد بيانات مسجلة لهذا اليوم حتى الآن.
      </div>

      <div v-if="restOfUsers.length > 0" class="bg-slate-800/40 rounded-xl border border-slate-700 overflow-hidden">
        <div class="max-h-96 overflow-y-auto custom-scrollbar">
          <ul class="divide-y divide-slate-700/50">
            <li v-for="(user, index) in restOfUsers" :key="user.alias"
              class="flex items-center justify-between p-4 hover:bg-slate-700/30 transition-colors">
              <div class="flex items-center gap-4">
                <span class="text-slate-500 font-bold w-6 text-center">{{ index + 4 }}</span>
                <span class="font-bold text-slate-200 tracking-wider flex items-center gap-2">
                  {{ user.alias }}
                  <span class="material-symbols-outlined text-sm text-slate-400">person</span>
                </span>
              </div>
              <span class="font-bold text-ramadan-gold">{{ user.totalPoints }} نقطة</span>
            </li>
          </ul>
        </div>
      </div>

    </div>

    <a href="https://wa.me/963969871572" target="_blank" rel="noopener noreferrer"
      class="fixed bottom-6 left-6 z-50 bg-[#25D366] hover:bg-[#1ebd5a] text-white p-3 md:p-4 rounded-full shadow-[0_4px_15px_rgba(37,211,102,0.4)] transition-all duration-300 hover:scale-110 flex items-center justify-center group"
      title="تواصل معنا عبر الواتساب">
      <svg xmlns="http://www.w3.org/2000/svg" width="28" height="28" fill="currentColor" viewBox="0 0 16 16">
        <path
          d="M13.601 2.326A7.85 7.85 0 0 0 7.994 0C3.627 0 .068 3.558.064 7.926c-.003 1.396.366 2.76 1.057 3.965L0 16l4.204-1.102a7.9 7.9 0 0 0 3.79.965h.004c4.368 0 7.926-3.558 7.93-7.93A7.9 7.9 0 0 0 13.6 2.326zM7.994 14.521a6.6 6.6 0 0 1-3.356-.92l-.24-.144-2.494.654.666-2.433-.156-.251a6.56 6.56 0 0 1-1.007-3.505c0-3.626 2.957-6.584 6.591-6.584a6.56 6.56 0 0 1 4.66 1.931 6.56 6.56 0 0 1 1.928 4.66c-.004 3.639-2.961 6.592-6.592 6.592m3.615-4.934c-.197-.099-1.17-.578-1.353-.646-.182-.065-.315-.099-.445.099-.133.197-.513.646-.627.775-.114.133-.232.148-.43.05-.197-.1-.836-.308-1.592-.985-.59-.525-.985-1.175-1.103-1.372-.114-.198-.011-.304.088-.403.087-.088.197-.232.296-.346.1-.114.133-.198.198-.33.065-.134.034-.248-.015-.347-.05-.099-.445-1.076-.612-1.47-.16-.389-.323-.335-.445-.34-.114-.007-.247-.007-.38-.007a.73.73 0 0 0-.529.247c-.182.198-.691.677-.691 1.654s.71 1.916.81 2.049c.098.133 1.394 2.132 3.383 2.992.47.205.84.326 1.129.418.475.152.904.129 1.246.08.38-.058 1.171-.48 1.338-.943.164-.464.164-.86.114-.943-.049-.084-.182-.133-.38-.232" />
      </svg>
    </a>

  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

// --- الحالة (State) ---
const allRecords = ref([]); // سيحفظ جميع البيانات القادمة من جوجل شيت
const loading = ref(true);
const error = ref(null);
const selectedDay = ref('all'); // الفلتر الافتراضي

// ضع رابط الـ Web App الخاص بك هنا (تأكد أنه محدث للنسخة الجديدة)
const API_URL = 'https://script.google.com/macros/s/AKfycbwRZEMpFREMjYoMgsylcqFrl7InkSdnFNStgEJUSnoGklfle_OSnvR6Y-_hftlKOzWD/exec';

// --- الخصائص المحسوبة (Computed) ---

// 1. استخراج الأيام المتاحة فريدة (بدون تكرار) لوضعها في القائمة المنسدلة
const availableDays = computed(() => {
  const daysSet = new Set(allRecords.value.map(record => record.day));
  return Array.from(daysSet);
});

// 2. فلترة وتجميع النقاط بناءً على اليوم المختار
const aggregatedUsers = computed(() => {
  const userTotals = {};

  allRecords.value.forEach(record => {
    // إذا كان هناك يوم محدد (ليس "الكل")، قم بتخطي الأيام الأخرى
    if (selectedDay.value !== 'all' && record.day !== selectedDay.value) {
      return;
    }

    // جمع النقاط
    if (userTotals[record.alias]) {
      userTotals[record.alias] += record.points;
    } else {
      userTotals[record.alias] = record.points;
    }
  });

  // تحويل الكائن إلى مصفوفة
  return Object.keys(userTotals).map(alias => ({
    alias: alias,
    totalPoints: userTotals[alias]
  }));
});

// 3. ترتيب المستخدمين المجمعين تنازلياً
const sortedUsers = computed(() => {
  return [...aggregatedUsers.value].sort((a, b) => b.totalPoints - a.totalPoints);
});

// 4. استخراج أول 3
const topThree = computed(() => sortedUsers.value.slice(0, 3));

// 5. استخراج الباقي
const restOfUsers = computed(() => sortedUsers.value.slice(3));

// --- الدوال (Methods) ---
const fetchLeaderboardData = async () => {
  loading.value = true;
  error.value = null;

  try {
    const response = await fetch(API_URL);
    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);

    // الآن الـ API يعيد مصفوفة مفصلة لكل العمليات
    allRecords.value = await response.json();

  } catch (err) {
    console.error("Failed to fetch leaderboard:", err);
    error.value = "فشل في تحميل البيانات، يرجى المحاولة لاحقاً.";
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  fetchLeaderboardData();
});
</script>

<style scoped>
/* تنسيق شريط التمرير ليناسب الألوان */
.custom-scrollbar::-webkit-scrollbar {
  width: 6px;
}

.custom-scrollbar::-webkit-scrollbar-track {
  background: rgba(30, 41, 59, 0.5);
}

.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgba(71, 85, 105, 0.8);
  border-radius: 10px;
}

.custom-scrollbar::-webkit-scrollbar-thumb:hover {
  background: rgba(251, 191, 36, 0.5);
  /* رمضان جولد عند التمرير */
}
</style>