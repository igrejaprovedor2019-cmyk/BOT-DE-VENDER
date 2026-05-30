require("dotenv").config();

const {
  Client,
  GatewayIntentBits,
  Partials,
  EmbedBuilder,
  ActionRowBuilder,
  ButtonBuilder,
  ButtonStyle,
  StringSelectMenuBuilder,
  ModalBuilder,
  TextInputBuilder,
  TextInputStyle,
  PermissionsBitField,
  ChannelType
} = require("discord.js");

const mercadopago = require("mercadopago");
const QRCode = require("qrcode");
const fs = require("fs-extra");

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMessages,
    GatewayIntentBits.MessageContent
  ],
  partials: [Partials.Channel]
});

const configPath = "./config.json";
const estoquePath = "./estoque.json";

if (!fs.existsSync(configPath)) {
  fs.writeJsonSync(configPath, {
    nome: "Painel",
    descricao: "Configure o painel",
    imagem: "",
    pix: "",
    token: ""
  });
}

if (!fs.existsSync(estoquePath)) {
  fs.writeJsonSync(estoquePath, []);
}

function getConfig() {
  return fs.readJsonSync(configPath);
}

function getEstoque() {
  return fs.readJsonSync(estoquePath);
}

client.on("ready", () => {
  console.log(`${client.user.tag} online`);
});

client.on("messageCreate", async (message) => {
  if (message.author.bot) return;

  if (message.content === ".vendas") {

    const config = getConfig();
    const produtos = getEstoque();

    const embed = new EmbedBuilder()
      .setColor("#6a00ff")
      .setTitle(config.nome)
      .setDescription(config.descricao);

    if (config.imagem) embed.setImage(config.imagem);

    const row1 = new ActionRowBuilder().addComponents(
      new ButtonBuilder()
        .setCustomId("configurar")
        .setLabel("⚙ Configurar")
        .setStyle(ButtonStyle.Secondary),

      new ButtonBuilder()
        .setCustomId("estoque")
        .setLabel("➕ Adicionar Estoque")
        .setStyle(ButtonStyle.Success)
    );

    const menu = new StringSelectMenuBuilder()
      .setCustomId("produtos")
      .setPlaceholder("Selecione um Produto");

    produtos.forEach((p, i) => {
      menu.addOptions({
        label: p.nome,
        description: `R$ ${p.valor}`,
        value: `${i}`
      });
    });

    const row2 = new ActionRowBuilder().addComponents(menu);

    message.channel.send({
      embeds: [embed],
      components: [row1, row2]
    });
  }
});

client.on("interactionCreate", async (interaction) => {

  // CONFIGURAÇÃO
  if (interaction.isButton() && interaction.customId === "configurar") {

    const modal = new ModalBuilder()
      .setCustomId("modalConfig")
      .setTitle("Configurar Painel");

    const nome = new TextInputBuilder()
      .setCustomId("nome")
      .setLabel("Nome do Painel")
      .setStyle(TextInputStyle.Short);

    const descricao = new TextInputBuilder()
      .setCustomId("descricao")
      .setLabel("Descrição")
      .setStyle(TextInputStyle.Paragraph);

    const imagem = new TextInputBuilder()
      .setCustomId("imagem")
      .setLabel("URL da Imagem")
      .setStyle(TextInputStyle.Short);

    const pix = new TextInputBuilder()
      .setCustomId("pix")
      .setLabel("Chave Pix")
      .setStyle(TextInputStyle.Short);

    const token = new TextInputBuilder()
      .setCustomId("token")
      .setLabel("Token Mercado Pago")
      .setStyle(TextInputStyle.Short);

    modal.addComponents(
      new ActionRowBuilder().addComponents(nome),
      new ActionRowBuilder().addComponents(descricao),
      new ActionRowBuilder().addComponents(imagem),
      new ActionRowBuilder().addComponents(pix),
      new ActionRowBuilder().addComponents(token)
    );

    return interaction.showModal(modal);
  }

  // SALVAR CONFIG
  if (interaction.isModalSubmit() && interaction.customId === "modalConfig") {

    const data = {
      nome: interaction.fields.getTextInputValue("nome"),
      descricao: interaction.fields.getTextInputValue("descricao"),
      imagem: interaction.fields.getTextInputValue("imagem"),
      pix: interaction.fields.getTextInputValue("pix"),
      token: interaction.fields.getTextInputValue("token")
    };

    fs.writeJsonSync(configPath, data);

    interaction.reply({
      content: "Painel configurado.",
      ephemeral: true
    });
  }

  // ADICIONAR ESTOQUE
  if (interaction.isButton() && interaction.customId === "estoque") {

    const modal = new ModalBuilder()
      .setCustomId("modalEstoque")
      .setTitle("Adicionar Produto");

    const nome = new TextInputBuilder()
      .setCustomId("nome")
      .setLabel("Nome Produto")
      .setStyle(TextInputStyle.Short);

    const valor = new TextInputBuilder()
      .setCustomId("valor")
      .setLabel("Valor")
      .setStyle(TextInputStyle.Short);

    const estoque = new TextInputBuilder()
      .setCustomId("estoque")
      .setLabel("Estoque")
      .setPlaceholder("login:senha")
      .setStyle(TextInputStyle.Paragraph);

    modal.addComponents(
      new ActionRowBuilder().addComponents(nome),
      new ActionRowBuilder().addComponents(valor),
      new ActionRowBuilder().addComponents(estoque)
    );

    return interaction.showModal(modal);
  }

  // SALVAR ESTOQUE
  if (interaction.isModalSubmit() && interaction.customId === "modalEstoque") {

    const produtos = getEstoque();

    produtos.push({
      nome: interaction.fields.getTextInputValue("nome"),
      valor: interaction.fields.getTextInputValue("valor"),
      estoque: interaction.fields
        .getTextInputValue("estoque")
        .split("\n")
    });

    fs.writeJsonSync(estoquePath, produtos);

    interaction.reply({
      content: "Produto adicionado.",
      ephemeral: true
    });
  }

  // SELECIONAR PRODUTO
  if (interaction.isStringSelectMenu() && interaction.customId === "produtos") {

    const produtos = getEstoque();
    const produto = produtos[interaction.values[0]];

    const canal = await interaction.guild.channels.create({
      name: `carrinho-${interaction.user.username}`,
      type: ChannelType.GuildText,
      permissionOverwrites: [
        {
          id: interaction.guild.id,
          deny: [PermissionsBitField.Flags.ViewChannel]
        },
        {
          id: interaction.user.id,
          allow: [PermissionsBitField.Flags.ViewChannel]
        }
      ]
    });

    const embed = new EmbedBuilder()
      .setColor("#6a00ff")
      .setTitle(produto.nome)
      .setDescription(`Valor: R$ ${produto.valor}`);

    const row = new ActionRowBuilder().addComponents(
      new ButtonBuilder()
        .setCustomId(`pagar_${interaction.values[0]}`)
        .setLabel("💸 Pagar")
        .setStyle(ButtonStyle.Success)
    );

    canal.send({
      content: `${interaction.user}`,
      embeds: [embed],
      components: [row]
    });

    interaction.reply({
      content: `Carrinho criado: ${canal}`,
      ephemeral: true
    });

    setTimeout(() => {
      if (canal.deletable) canal.delete().catch(() => {});
    }, 600000);
  }

  // PAGAR
  if (interaction.isButton() && interaction.customId.startsWith("pagar_")) {

    const id = interaction.customId.split("_")[1];
    const produtos = getEstoque();
    const produto = produtos[id];
    const config = getConfig();

    mercadopago.configure({
      access_token: config.token
    });

    const payment_data = {
      transaction_amount: Number(produto.valor),
      description: produto.nome,
      payment_method_id: "pix",
      payer: {
        email: "cliente@gmail.com"
      }
    };

    const pagamento = await mercadopago.payment.create(payment_data);

    const qr = pagamento.body.point_of_interaction.transaction_data.qr_code;
    const qrimg = pagamento.body.point_of_interaction.transaction_data.qr_code_base64;

    const embed = new EmbedBuilder()
      .setColor("#6a00ff")
      .setTitle("Pagamento Pix")
      .setDescription(`
Valor: R$ ${produto.valor}

Copie e cole:
\`\`\`
${qr}
\`\`\`

Você possui 10 minutos para pagar.
`);

    await interaction.reply({
      embeds: [embed]
    });

    // VERIFICAÇÃO
    const intervalo = setInterval(async () => {

      const status = await mercadopago.payment.findById(
        pagamento.body.id
      );

      if (status.body.status === "approved") {

        clearInterval(intervalo);

        const item = produto.estoque.shift();

        fs.writeJsonSync(estoquePath, produtos);

        const aprovado = new EmbedBuilder()
          .setColor("Green")
          .setTitle("Pagamento Aprovado")
          .setDescription(`
Você receberá seu produto daqui um momento.
`);

        const row = new ActionRowBuilder().addComponents(
          new ButtonBuilder()
            .setCustomId("fechar")
            .setLabel("🗑 Excluir Canal")
            .setStyle(ButtonStyle.Danger)
        );

        await interaction.channel.send({
          embeds: [aprovado],
          components: [row]
        });

        await interaction.user.send(`
Seu produto:
${item}
`);

      }

    }, 5000);
  }

  // FECHAR
  if (interaction.isButton() && interaction.customId === "fechar") {

    if (!interaction.member.roles.cache.has("1510007036795162664")) {
      return interaction.reply({
        content: "Sem permissão.",
        ephemeral: true
      });
    }

    interaction.reply({
      content: "Excluindo...",
      ephemeral: true
    });

    setTimeout(() => {
      interaction.channel.delete().catch(() => {});
    }, 2000);
  }

});

client.login(process.env.TOKEN);
