<template>
  <div class="container">
    <h1>そうじ当番</h1>

    <!-- メンバー登録セクション -->
    <div class="input-section" v-if="!groupsGenerated && !isLoading">
      <div class="input-group">
        <input
          type="text"
          v-model="newMemberName"
          @keyup.enter="addMember"
          placeholder="名前を入力してください"
        />
        <button class="btn-primary" @click="addMember">追加</button>
      </div>

      <div class="member-list" v-if="members.length > 0">
        <div
          class="member-item"
          v-for="(member, index) in members"
          :key="index"
        >
          <span class="member-name">{{ member }}</span>
          <button class="btn-remove" @click="removeMember(index)">削除</button>
        </div>
      </div>

      <button
        class="btn-success"
        @click="startGrouping"
        :disabled="members.length < 2"
        v-if="members.length >= 2"
      >
        スタート
      </button>
    </div>

    <!-- ローディングアニメーション -->
    <div class="loading-section" v-if="isLoading">
      <div class="smiley-container">
        <div class="smiley">😊</div>
      </div>
      <p class="loading-text">組み合わせを考えています...</p>
    </div>

    <!-- グループ表示セクション -->
    <div
      class="groups-section"
      v-if="groupsGenerated && !isLoading"
      ref="groupsSection"
    >
      <h2>組み合わせ結果</h2>
      <div class="groups-grid">
        <div class="group-card" v-for="(group, index) in groups" :key="index">
          <div class="group-members">
            <div v-for="(member, idx) in group.members" :key="idx">
              {{ member }}
            </div>
          </div>
        </div>
      </div>

      <button class="btn-download" @click="downloadScreenshot">
        スクリーンショットをダウンロード
      </button>
      <button
        class="btn-primary"
        @click="reset"
        style="width: 100%; margin-top: 10px"
      >
        最初に戻る
      </button>
    </div>

    <div
      class="empty-state"
      v-if="members.length === 0 && !groupsGenerated && !isLoading"
    >
      メンバーを追加してください（最低2名必要です）
    </div>
  </div>
</template>

<script>
import {ref} from "vue";
import html2canvas from "html2canvas";

export default {
  name: "App",
  setup() {
    const members = ref([]);
    const newMemberName = ref("");
    const groups = ref([]);
    const groupsGenerated = ref(false);
    const isLoading = ref(false);
    const groupsSection = ref(null);

    const addMember = () => {
      const name = newMemberName.value.trim();
      if (name && !members.value.includes(name)) {
        members.value.push(name);
        newMemberName.value = "";
      }
    };

    const removeMember = (index) => {
      members.value.splice(index, 1);
    };

    const startGrouping = async () => {
      if (members.value.length < 2) {
        return;
      }

      // ローディング開始
      isLoading.value = true;
      groupsGenerated.value = false;

      // 5秒待機（ローディングアニメーション表示）
      await new Promise((resolve) => setTimeout(resolve, 5000));

      // 組み合わせ生成
      generateGroups();

      // ローディング終了
      isLoading.value = false;
      groupsGenerated.value = true;
    };

    const generateGroups = () => {
      const shuffled = [...members.value].sort(() => Math.random() - 0.5);
      const pairs = [];

      // 偶数人数の場合
      if (shuffled.length % 2 === 0) {
        // 2人ずつペアにする
        for (let i = 0; i < shuffled.length; i += 2) {
          pairs.push({
            members: [shuffled[i], shuffled[i + 1]],
          });
        }
      } else {
        // 奇数人数の場合
        // まず、全員を2人ずつペアにする（最後の1人は余る）
        for (let i = 0; i < shuffled.length - 1; i += 2) {
          pairs.push({
            members: [shuffled[i], shuffled[i + 1]],
          });
        }

        // 最後の1人を発表（1人だけのグループとして表示）
        const extraPerson = shuffled[shuffled.length - 1];
        pairs.push({
          members: [extraPerson],
        });

        // 最後の一人以外の誰かをランダムで選出して最後のグループに追加
        // 選ばれた人は2回登場する（1回は元のペア、1回は最後のグループ）
        // 同じグループに同じ人が2回登場してはいけないので、元のペアから選ばれた人を削除する
        const allOtherMembers = shuffled.slice(0, -1); // 最後の1人以外
        const randomIndex = Math.floor(Math.random() * allOtherMembers.length);
        const randomPerson = allOtherMembers[randomIndex];

        // 選ばれた人を最後のグループに追加（元のペアからは削除しない）
        // 選ばれた人は2回登場する（1回は元のペア、1回は最後のグループ）
        pairs[pairs.length - 1].members.push(randomPerson);
      }

      groups.value = pairs;
    };

    const downloadScreenshot = async () => {
      if (!groupsSection.value) return;

      try {
        const canvas = await html2canvas(groupsSection.value, {
          scale: 2,
          backgroundColor: "#ffffff",
        });

        const link = document.createElement("a");
        link.download = "そうじ当番_組み合わせ結果.png";
        link.href = canvas.toDataURL("image/png");
        link.click();
      } catch (error) {
        console.error("スクリーンショット生成エラー:", error);
        alert("スクリーンショットの生成に失敗しました");
      }
    };

    const reset = () => {
      members.value = [];
      groups.value = [];
      groupsGenerated.value = false;
      isLoading.value = false;
      newMemberName.value = "";
    };

    return {
      members,
      newMemberName,
      groups,
      groupsGenerated,
      isLoading,
      groupsSection,
      addMember,
      removeMember,
      startGrouping,
      downloadScreenshot,
      reset,
    };
  },
};
</script>
