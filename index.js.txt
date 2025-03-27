const { Client, GatewayIntentBits, ActionRowBuilder, StringSelectMenuBuilder } = require("discord.js");
const axios = require("axios");

// BotトークンとGASのWebhook URL
const DISCORD_TOKEN = process.env.DISCORD_TOKEN;
const GAS_WEBHOOK_URL = process.env.GAS_WEBHOOK_URL;

// Discordクライアント設定
const client = new Client({
  intents: [GatewayIntentBits.Guilds, GatewayIntentBits.GuildMessages, GatewayIntentBits.MessageContent],
});

// 業務内容リスト
const taskTypes = [
  "撮影内容考案", "パッケージ作成", "パッケージ作成(指示書)",
  "タレント打ち合わせ", "打ち合わせ", "撮影",
  "販促活動", "販売準備作業", "作品編集",
  "その他作業", "休憩"
];

// 担当者リスト
const staffList = ["田中太郎", "佐藤花子", "鈴木一郎", "山本美咲"];

client.once("ready", () => {
  console.log(`✅ Botがログインしました：${client.user.tag}`);
});

// メッセージで「/業務報告」と入力するとプルダウン表示
client.on("messageCreate", async (message) => {
  if (message.author.bot || message.content !== "/業務報告") return;

  // 担当者選択のプルダウン
  const staffSelect = new StringSelectMenuBuilder()
    .setCustomId("selectStaff")
    .setPlaceholder("担当者を選択")
    .addOptions(
      staffList.map((staff) => ({
        label: staff,
        value: staff,
      }))
    );

  // 業務内容選択のプルダウン
  const taskSelect = new StringSelectMenuBuilder()
    .setCustomId("selectTask")
    .setPlaceholder("業務内容を選択")
    .addOptions(
      taskTypes.map((task) => ({
        label: task,
        value: task,
      }))
    );

  const row1 = new ActionRowBuilder().addComponents(staffSelect);
  const row2 = new ActionRowBuilder().addComponents(taskSelect);

  await message.reply({
    content: "担当者と業務内容を選択してください",
    components: [row1, row2],
  });
});

// プルダウン選択後の処理
client.on("interactionCreate", async (interaction) => {
  if (!interaction.isStringSelectMenu()) return;

  const selectedStaff = interaction.customId === "selectStaff" ? interaction.values[0] : null;
  const selectedTask = interaction.customId === "selectTask" ? interaction.values[0] : null;

  if (selectedStaff && selectedTask) {
    // Google Apps Script に送信
    await axios.post(GAS_WEBHOOK_URL, {
      username: selectedStaff,
      task_type: selectedTask,
      task_detail: "詳細未記入",
    });

    await interaction.reply({
      content: `✅ ${selectedStaff} が「${selectedTask}」を記録しました！`,
      ephemeral: true,
    });
  }
});

// Botログイン
client.login(DISCORD_TOKEN);