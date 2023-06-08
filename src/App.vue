<template>
  <v-app>
    <v-app-bar app color="blue" image="src/assets/background.jpeg">
      <v-app-bar-nav-icon>
        <v-icon @click="toggleSidebar">mdi-menu</v-icon>
      </v-app-bar-nav-icon>

      <v-toolbar-title>
        <span class="mr-2">Logo</span>
        <span>凭证监管</span>
      </v-toolbar-title>

      <v-spacer></v-spacer>

      <v-btn-toggle v-model="selectedLanguage" color="black">
        <v-btn :value="'en'" text="English">English</v-btn>
        <v-btn :value="'zh'" text="中文">中文</v-btn>
      </v-btn-toggle>

      <v-btn icon @click="toggleExpand">
        <v-icon>{{ isExpanded ? 'mdi-chevron-up' : 'mdi-chevron-down' }}</v-icon>
      </v-btn>
    </v-app-bar>

    <v-navigation-drawer v-model="isSidebarOpen" app image="src/assets/background.jpeg">
      <!-- 侧边栏内容 -->
    </v-navigation-drawer>


    <div class="content-wrapper background-container" >
      <v-main >
        <v-container class="form-shadow">
          <v-row justify="center" cols="12" sm="6" md="4" lg="3" v-for="line in lines" :key="line.id"
                 class="align-middle">
            <v-col>
              <v-card class="v-card" :color="line.color">
                <v-card-actions>
                  <v-checkbox-btn v-model="line.checked" class="checkbox-style" color="pink"></v-checkbox-btn>
                  <v-btn @click="openDialog(line)">
                    <span class="align-middle font-size">{{ line.text }}</span>
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-col>
          </v-row>
          <v-row justify="center">
            <v-col cols="12" sm="6" md="4" lg="3">
              <v-btn @click="pushData" :disabled="!isAnyLineChecked" class="mx-auto"
                     :color="isAnyLineChecked ? 'blue' : ''"
                     width="500"
                     height="40"
                     :class="{ 'grey lighten-2': !isAnyLineChecked }">
                发布凭证
              </v-btn>
            </v-col>
          </v-row>
        </v-container>
      </v-main>

      <!-- 添加对话框组件 -->
      <v-dialog v-model="dialogVisible" max-width="600px">
        <v-card style="border-radius: 20px;color:#3F3F3F">
          <v-card-title>
            <span class="headline">{{ dialogLine.text }}</span>
          </v-card-title>
          <v-card-text>
            <!-- 发票表格数据 -->
            <template v-if="dialogLine.text === '发票'">
              <v-text-field v-model="dialogLine.tableData.title" label="标题"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.invoiceNumber" label="发票编号"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.invoiceRecipient" label="发票抬头"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.invoiceContent" label="发票内容"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.invoiceAmount" label="发票金额"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.invoiceDate" label="发票日期"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.sellerInformation" label="销售方信息"></v-text-field>
            </template>

            <!-- 收据表格数据 -->
            <template v-else-if="dialogLine.text === '收据'">
              <v-text-field v-model="dialogLine.tableData.title" label="标题"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.receiptNumber" label="收据编号"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.recipient" label="收款方"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.paymentDetails" label="收款内容"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.receiptAmount" label="收据金额"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.receiptDate" label="收据日期"></v-text-field>
            </template>

            <!-- 收据表格数据 -->
            <template v-else-if="dialogLine.text === '会议凭证'">
              <v-text-field v-model="dialogLine.tableData.title" label="标题"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.venue" label="会议地点"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.date" label="会议日期"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.participants" label="参会人员"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.agenda" label="会议议程"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.remarks" label="备注"></v-text-field>
            </template>

            <!-- 收据表格数据 -->
            <template v-else-if="dialogLine.text === '门票'">
              <v-text-field v-model="dialogLine.tableData.title" label="标题"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.ticketType" label="门票类型"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.ticketNumber" label="门票编号"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.purchaserName" label="购票人姓名"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.purchaseDate" label="购票日期"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.ticketPrice" label="票价"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.ticketValidity" label="门票有效期"></v-text-field>
              <v-text-field v-model="dialogLine.tableData.remarks" label="备注"></v-text-field>
            </template>

            <!-- 添加更多行的输入字段 -->

          </v-card-text>

          <v-card-actions>
            <v-btn color="blue" @click="saveDialog">保存</v-btn>
            <v-btn @click="closeDialog">取消</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!--      添加提醒-->
      <v-dialog class="remind" v-model="remindVisible" max-width="600px">
        <v-card>
          <v-card-title>
            <span class="headline">{{ remindLine.model }}</span>
          </v-card-title>
          <v-card-text>
            <span>{{ remindLine.text }}</span>
          </v-card-text>
          <v-card-actions>
            <v-btn color="primary" @click="saveRemind">确认</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

    </div>
    <v-dialog v-model="isShow" max-width="600px">
      <canvas class="canvas" ref="qrImage"></canvas>
      <v-btn color="blue" @click="closeImage">确认</v-btn>
    </v-dialog>

  </v-app>
</template>


<script>

import QRCode from 'qrcode';
import {tr} from "vuetify/locale";


async function generateQRCode(text) {
  try {
    return await QRCode.toDataURL(text);
  } catch (error) {
    console.error('无法生成二维码:', error);
    return null;
  }
}

export default {
  data() {
    return {
      canvas: null,
      isSidebarOpen: false,
      selectedLanguage: 'zh',
      showCertificateDialog: false,
      isExpanded: false,
      isSaving: false,
      lines: [
        {
          id: 1,
          text: '发票',
          tableData: {
            title: null,
            invoiceNumber: null,
            invoiceRecipient: null,
            invoiceContent: null,
            invoiceAmount: null,
            invoiceDate: null,
            sellerInformation: null
          },
          qrCode: null,
          checked: false,
          color: '#E4DAD2'
        },
        {
          id: 2,
          text: '收据',
          tableData: {
            title: null,
            receiptNumber: null,
            recipient: null,
            paymentDetails: null,
            receiptAmount: null,
            receiptDate: null
          },
          qrCode: null,
          checked: false,
          color: '#B2B5FD'
        },
        {
          id: 3,
          text: '会议凭证',
          tableData: {
            title: null,
            venue: null,
            date: null,
            participants: null,
            agenda: null,
            remarks: null
          },
          qrCode: null,
          checked: false,
          color: '#B2DDFD'
        },
        {
          id: 4,
          text: '门票',
          tableData: {
            title: null,
            ticketType: null,
            ticketNumber: null,
            purchaserName: null,
            purchaseDate: null,
            ticketPrice: null,
            ticketValidity: null,
            remarks: null
          },
          qrCode: null,
          checked: false,
          color: '#D1ACFF'
        },

      ],
      dialogVisible: false, // 对话框的显示状态
      dialogLine: null,
      isShow: false,
      remindVisible: false,
      remindLine: {text: null, model: null},
    };
  },
  computed: {
    isAnyLineChecked() {
      return this.lines.some(line => line.checked);
    }
  },
  mounted() {
    // this.canvas = this.$refs.qrImage;
  },
  methods: {
    toggleSidebar() {
      this.isSidebarOpen = !this.isSidebarOpen;
    },
    toggleExpand() {
      this.isExpanded = !this.isExpanded;
    },
    openDialog(line) {
      this.dialogLine = line; // 设置对话框当前行对象
      this.dialogVisible = true; // 打开对话框
      console.log(this.dialogLine)
    },
    saveDialog() {

      this.closeDialog(); // 保存完成后关闭对话框
    },
    closeDialog() {
      this.dialogVisible = false; // 关闭对话框
    },
    closeImage() {
      this.isShow = false;
    },
    saveRemind() {
      this.remindVisible = false;
    },
    async pushData() {
      this.isShow = true;
      let canvas;
      await this.$nextTick(() => {
        canvas = this.$refs.qrImage;
      })

      const line = this.lines.find(line => line.checked === true);
      console.log(line);
      let qrCodeData = null;
      if (line) {
        qrCodeData = await generateQRCode(JSON.stringify(line));
      } else {
        this.remindLine.model = "提醒";
        this.remindLine.text = "未勾选";
        this.remindVisible = true;
        return;
      }
      if (qrCodeData) {
        console.log(canvas);
        canvas.width = 300;
        canvas.height = 300;

        const ctx = canvas.getContext('2d');
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        const qrCodeImage = new Image();
        qrCodeImage.src = qrCodeData;
        qrCodeImage.onload = () => {
          ctx.drawImage(qrCodeImage, 0, 0, canvas.width, canvas.height);
        };

      } else {
        this.remindLine.model = "错误";
        this.remindLine.text = "无法生成二维码";
        this.remindVisible = true;
        return;
      }
    },
  },
};
</script>


<style>
body{
  background-image: url("/src/assets/image.png") !important;
  background-size: cover !important;
  background-repeat: no-repeat !important;
}


.checkbox-style .v-input--selection-controls__ripple {
  border: 2px solid currentColor;
}

.canvas {
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 调整内容居中显示 */
.content-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

/* 自定义卡片样式 */
.v-card {
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.font-size {
  font-size: 18px; /* 调整字体大小 */
  font-weight: bold;
}

.align-middle {
  display: inline-flex;
  align-items: center;
}


/* 调整响应式布局 */
@media only screen and (max-width: 600px) {
  .v-row {
    flex-direction: column;
  }

  .v-col {
    margin-bottom: 16px;
  }
}

.align-middle {
  display: inline-flex;
  align-items: center;
}

.align-middle {
  display: inline-flex;
  align-items: center;
}

.v-card {
  border: none;
  box-shadow: none;
}

.mx-auto {
  margin-left: auto;
  margin-right: auto;
  width: 300px;
  color: #535bf2;
}

.remind {

}

.form-shadow {
  box-shadow: 1px 2px 40px rgba(0, 0, 0, 0.1);
}

.background-container {
  background-image: url("/src/assets/image.png") !important;
  background-size: 360% !important;
  background-position-x: -690px !important;
  background-position-y: -35px !important;
  background-repeat: no-repeat !important;

}
</style>
