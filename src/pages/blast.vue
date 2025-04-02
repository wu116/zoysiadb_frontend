<script lang="ts" setup>
import axios from 'axios'
import { computed, ref, watch, onMounted } from 'vue'
import { ElLoading } from 'element-plus'

axios.defaults.xsrfHeaderName = "X-CSRFToken";
axios.defaults.xsrfCookieName = "csrftoken";
axios.defaults.withCredentials = true;  // 允许携带Cookie

// 定义数据框
const textarea1 = ref('')
const textarea1rules = {
  textarea1: [
    {
      validator: (rule: any, value: string, callback: any) => {
        const lines = value.split('\n')
        if (!lines[0]?.startsWith('>')) {
          callback(new Error('第一行必须以 ">" 开头'))
        } else if (lines[0].length >= 26) {
          callback(new Error('序列名必须少于25个字符'))
        } else if (!lines.slice(1).every(line => /^[A-Za-z]+$/.test(line.trim()))) {
          callback(new Error('第二行及后续行只能包含字母'))
        } else {
          callback()
        }
      },
      trigger: 'blur'
    }
  ]
}

// 定义数据库选项
// const DBvalue = ref('CDS')
// const DBoptions = [
//   {
//     DBvalue: 'genome',
//     label: 'genome',
//   },
//   {
//     DBvalue: 'mRNA',
//     label: 'mRNA',
//   },
//   {
//     DBvalue: 'CDS',
//     label: 'CDS',
//   },
//   {
//     DBvalue: 'protein',
//     label: 'protein',
//   },
// ]
const DBvalue = ref('');
const dbOptions = ref<
  { label: string; options: { DBvalue: string; label: string }[] }[]
>([]);
const dbTypeMap = ref<Record<string, string>>({});
const fetchDBOptions = async () => {
  try {
    const response = await axios.get('/api/blastapi/get_blastdb/');
    // 将数据映射为下拉选项格式
    const groups: Record<string, { DBvalue: string; label: string }[]> = {};
    const typeMap: Record<string, string> = {};
    response.data.forEach((item: { file_type: string; file_name: string }) => {
      if (!groups[item.file_type]) {
        groups[item.file_type] = [];
      }
      groups[item.file_type].push({
        DBvalue: item.file_name,
        label: item.file_name
      });
      typeMap[item.file_name] = item.file_type;
    });

    dbOptions.value = Object.entries(groups).map(([type, options]) => ({
      label: type,
      options: options
    }));
    dbTypeMap.value = typeMap;
  } catch (error) {
    console.error('获取数据库选项失败:', error);
  }
};

// 初始化时加载
onMounted(() => {
  fetchDBOptions().then(() => {
    if (dbOptions.value.length > 0 && dbOptions.value[0].options.length > 0) {
      DBvalue.value = dbOptions.value[0].options[0].DBvalue;
    }
  });
});

// 定义比对软件选项
const Programvalue = ref('blastn')
const Programoptions = [
  {
    Programvalue: 'blastn',
    label: 'blastn',
  },
  {
    Programvalue: 'blastp',
    label: 'blastp',
  },
  {
    Programvalue: 'blastx',
    label: 'blastx',
  },
  {
    Programvalue: 'tblastn',
    label: 'tblastn',
  },
  {
    Programvalue: 'tblastx',
    label: 'tblastx',
  },
]
const avProgramoptions = computed(() => {
  const dbType = dbTypeMap.value[DBvalue.value];
  return Programoptions.map(option => {
    if (dbType === 'protein') {
      // 当数据库为 protein 时，只允许 blastp 和 tblastn
      return {
        ...option,
        disabled: !['blastp', 'tblastn'].includes(option.Programvalue)
      }
    } else {
      // 非 protein 数据库时，禁用 blastp 和 tblastn
      return {
        ...option,
        disabled: ['blastp', 'tblastn'].includes(option.Programvalue)
      }
    }
  })
})

// 监听数据库变化，自动清除非法选项
watch(DBvalue, (newDB) => {
  const dbType = dbTypeMap.value[newDB];
  const allowedPrograms = dbType === 'protein' 
    ? ['blastp', 'tblastn'] 
    : ['blastn', 'blastx', 'tblastx']
  
  if (!allowedPrograms.includes(Programvalue.value)) {
    Programvalue.value = allowedPrograms[0]; // 清除不符合条件的值
  }
})

// 定义evalue num
const num = ref(25)

// example事件
const handleexample = () => {
  const exampleHeader = ">Zj13G009140.mRNA1\n" // 公共的FASTA头部
  const dbType = dbTypeMap.value[DBvalue.value];
  const isProteinDB = dbType === 'protein'
  // 根据选择的程序类型设置不同示例序列
  textarea1.value = exampleHeader + (
    isProteinDB ? 
    `MTCLERKTMATDENAVGDRAAGENAVDHGNNHPGPGGMDLAGDGDHVPKTRKPYTITKQR
EKWTEDEHRRFLEALQLHGRAWRRIQEHIGTKTAVQIRSHAQKFFTKVVRESSGSNGGSG
AGAPSIQIPPPRPKRKPAHPYPRKVDGAAKKHAPTLKRLEKPPVRRMQSLREQDDGSPTS
VLTAAQTVLRADALDGAFSNTWSGGDRSRAPSVVGSDEHGNGGGSSVDREDGCLSPSIAA
AELEVKAPNTRAFCDAKDCAGSEASVFKLFGKKVAVKDSYQHLQNGRNLKIDVSPACVTK
PTGNAIPCAGANSWNHLWPGNMQQVMYFLPGPGSFPAQSVMPWLGYNGSMPCSLFYPQAV
TSNQEHHIPSESPDPEGSLTGSNTASSVAPASAAQNSDAVESRAEQGKASESVTKPAVKR
LLKCPSSASTNRRGFMPYKRCAAESDAPRSVAPGEEADGELTRLCL` :
`ATGACCTGCTTGGAGAGGAAAACGATGGCGACTGACGAGAACGCCGTTGGGGATCGCGCG
GCCGGTGAAAACGCCGTGGATCACGGCAACAATCATCCCGGCCCCGGCGGCATGGATTTG
GCAGGGGACGGCGACCACGTGCCCAAGACGAGGAAGCCGTACACGATCACGAAGCAGCGG
GAGAAGTGGACGGAGGACGAGCACAGGCGGTTCTTGGAGGCTCTGCAGTTACACGGCCGC
GCATGGCGCCGCATACAAGAGCACATCGGCACCAAGACCGCGGTGCAAATCCGGAGCCAC
GCGCAGAAGTTCTTCACCAAGGTCGTCAGAGAATCGTCGGGGAGCAACGGCGGCTCGGGC
GCCGGCGCCCCTTCGATCCAGATCCCTCCGCCGCGTCCCAAGAGGAAGCCGGCCCACCCG
TACCCGCGCAAGGTGGACGGCGCGGCCAAGAAGCACGCGCCGACGCTGAAGCGGCTGGAG
AAGCCGCCGGTCCGGCGGATGCAGTCTCTTCGCGAGCAGGACGACGGGTCGCCGACGTCC
GTGTTGACGGCGGCGCAGACGGTATTGCGGGCGGACGCGCTTGACGGTGCGTTCTCCAAC
ACCTGGAGCGGCGGCGACAGGTCGCGGGCTCCGTCAGTGGTCGGTTCAGATGAGCATGGC
AACGGAGGGGGCTCATCGGTGGACAGAGAGGACGGCTGCCTGTCGCCAAGCATAGCAGCT
GCTGAGCTCGAAGTGAAAGCGCCAAATACCAGGGCGTTCTGTGATGCAAAAGATTGTGCA
GGATCAGAAGCTTCAGTCTTCAAGCTGTTTGGAAAGAAAGTTGCGGTGAAGGATTCCTAC
CAGCACTTGCAGAACGGCAGAAACCTGAAAATAGATGTTTCACCTGCTTGTGTTACCAAG
CCAACCGGAAACGCAATTCCTTGTGCCGGCGCGAACTCCTGGAACCACCTATGGCCGGGC
AACATGCAGCAGGTTATGTACTTCCTTCCCGGACCAGGTAGTTTTCCTGCGCAGTCTGTC
ATGCCATGGTTGGGCTACAATGGGAGCATGCCTTGCTCCCTGTTCTACCCGCAGGCGGTG
ACCTCAAACCAGGAGCACCACATACCTTCAGAGTCTCCTGATCCAGAAGGGTCACTGACC
GGCTCGAACACGGCCTCCAGCGTCGCGCCGGCATCAGCCGCTCAGAATTCGGACGCCGTA
GAGTCCCGCGCCGAGCAAGGAAAAGCCAGTGAAAGCGTCACAAAGCCTGCTGTTAAACGA
CTGTTGAAGTGCCCGAGCTCGGCTTCTACGAACAGGAGAGGGTTCATGCCTTACAAGAGG
TGCGCGGCAGAGAGCGATGCACCGCGATCGGTGGCCCCCGGAGAGGAGGCAGACGGCGAG
CTGACGAGGCTGTGCTTGTAA`
  )
};

// 清除事件
const handleClear = () => {
  textarea1.value = ''; // 直接清空 textarea 内容
};

// 重置事件
const handleReset = () => {
  textarea1.value = ''; // 直接清空 textarea 内容
  // 重置数据库选择，确保取第一个分组的第一个选项
  if (dbOptions.value.length > 0 && dbOptions.value[0].options.length > 0) {
    DBvalue.value = dbOptions.value[0].options[0].DBvalue;
  }

  Programvalue.value = 'blastn'; // 重置比对软件选择
  num.value = 25; // 重置 evalue 值
};

// submit验证
const isSubmitEnabled = computed(() => {
  const lines = textarea1.value.split('\n')
  if (lines.length === 0) return false
  
  // 验证第一行以 ">" 开头
  const firstLineValid = lines[0].startsWith('>') && lines[0].length <= 25
  
  // 验证后续行均为纯字母（忽略空行）
  const bodyValid = lines.slice(1).every(line => 
    /^[A-Za-z]+$/.test(line.trim()) || line.trim() === ''
  )
  
  return firstLineValid && bodyValid && textarea1.value.trim() !== ''
})

// sumbit事件
const handleSubmit = async () => {
  try {
    // 开启 loading
    const loading = ElLoading.service({
      lock: true,
      text: 'Running BLAST, please wait...',
      background: 'rgba(0, 0, 0, 0.7)',
      fullscreen: true, 
    });
    
    // 分割数据
    const lines = textarea1.value.split('\n')
    const header = lines[0].replace('>', '').trim() // 提取 ">" 后的内容
    const body = lines.slice(1).filter(line => line.trim() !== '') // 过滤空行

    // 发送到后端（示例使用 axios）
    const blastresponse = await axios.post('/api/blastapi/blast/', {
      header: header,
      body: body.join('\n'), // 或直接传递数组
      db: DBvalue.value,
      program: Programvalue.value,
      evalue: `1e-${num.value}`
    })

    // 清空表单（可选）
    textarea1.value = ''
    console.log('Results:', blastresponse.data)
    setTimeout(() => {
      loading.close()
    }, 1000)

    // 1. 生成新页面 HTML 内容
    interface BlastResult {
      qseqid: string;
      sseqid: string;
      pident: number;
      length: number;
      qstart: number;
      qend: number;
      sstart: number;
      send: number;
      evalue: number;
      bitscore: number;
    }
    const results: BlastResult[] = blastresponse.data.results
    const htmlRows = results.map(item => `
      <tr>
        <td>${item.qseqid}</td>
        <td>${item.sseqid}</td>
        <td>${item.pident}</td>
        <td>${item.length}</td>
        <td>${item.qstart}</td>
        <td>${item.qend}</td>
        <td>${item.sstart}</td>
        <td>${item.send}</td>
        <td>${item.evalue}</td>
        <td>${item.bitscore}</td>
      </tr>
    `).join('')

    // 2. 打开新窗口并写入内容
    const newWindow = window.open('', '_blank')
    if(newWindow){
      const htmlContent = `
        <html>
          <head>
            <title>BLAST 结果</title>
            <style>
              body { font-family: Arial; padding: 20px; }
              table { border-collapse: collapse; width: 100%; }
              th, td { border: 1px solid #ccc; padding: 8px; text-align: left; }
              th { background-color: #f2f2f2; }
            </style>
          </head>
          <body>
            <h2>BLAST Search Results</h2>
            <table>
              <thead>
                <tr>
                  <th>Query ID</th>
                  <th>Subject ID</th>
                  <th>% Identity</th>
                  <th>alignment length</th>
                  <th>Query start</th>
                  <th>Query end</th>
                  <th>Subject start</th>
                  <th>Subject end</th>
                  <th>E-value</th>
                  <th>Bit Score</th>
                </tr>
              </thead>
              <tbody>
                ${htmlRows}
              </tbody>
            </table>
          </body>
        </html>
      `
      newWindow.document.write(htmlContent)
      newWindow.document.close()
    } else {
      console.error('Cannot open new Window, might be intercepted by browser.')
    }


  } catch (error) {
    if (error instanceof Error) {
      console.log(error.message);
    } else {
      console.log("Unknown error:", error);
    }
  }
}
</script>

<template>
  <div class="common-layout">
    <el-container>
      <div style="margin: 20px 0" />
      <el-header>
        <span style="font-size: xx-large;"><b>BLAST</b></span>
      </el-header>
      <el-container>
        <el-main>
          <!-- 选项 -->
          <el-row>
            <el-col :span="22" :offset="1">
              <div class="flex gap-4 items-center">
                <!-- 选择数据库 -->
                Database:
                <el-select v-model="DBvalue" placeholder="Select" style="width: 180px">
                  <el-option-group
                    v-for="group in dbOptions"
                    :key="group.label"
                    :label="group.label"
                  >
                    <el-option
                      v-for="option in group.options"
                      :key="option.DBvalue"
                      :label="option.label"
                      :value="option.DBvalue"
                    />
                  </el-option-group>
                </el-select>
                <!-- 选择比对软件 -->
                Program:
                <el-select v-model="Programvalue" placeholder="Select" style="width: 120px">
                  <el-option
                    v-for="item in avProgramoptions"
                    :key="item.Programvalue"
                    :label="item.label"
                    :value="item.Programvalue"
                    :disabled="item.disabled"
                  />
                </el-select>
                <!-- 指定evalue -->
                evalue:
                <el-input-number v-model="num" :min="0" :max="200">
                  <template #prefix>
                    <span>1e-</span>
                  </template>
                </el-input-number>
              </div>
            </el-col>
          </el-row>
          <div style="margin: 10px 0" />
          <!-- 框头 -->
          <el-row>
            <el-col :span="1" :offset="1">
              <div class="flex items-end">
                <span style="color: red;">*</span>FASTA:
                <el-button link type="primary" @click="handleexample">
                  example
                </el-button>
              </div>
            </el-col>
          </el-row>
          <!-- 输入框 -->
          <el-row>
            <el-col :span="18" :offset="1">
              <el-form :model="{ textarea1 }" :rules="textarea1rules">
                <el-form-item prop="textarea1">
                  <el-input
                    v-model="textarea1"
                    style="width: 100%"
                    :autosize="{ minRows: 8, maxRows: 10 }"
                    type="textarea"
                    placeholder=">seq1&#10;ATCGATCG..."
                  />
                </el-form-item>
              </el-form>
            </el-col>
          </el-row>
          <div style="margin: 10px 0" />
          <!-- 提交与清除按钮 -->
          <el-row>
            <el-col :span="4" :offset="1">
              <div class="flex items-center mb-4">
                <el-button plain :disabled="!isSubmitEnabled" @click="handleSubmit">
                  Submit
                </el-button>
                <el-button plain @click="handleClear">
                  Clear
                </el-button>
                <el-button plain @click="handleReset">
                  Reset
                </el-button>
              </div>
            </el-col>
          </el-row>
        </el-main>
        <!-- 侧栏备注 -->
        <el-aside width="20%">
          <div class="flex" style="text-align: left;">
            NOTE:
            <br>
            *.genome: entire assembly sequences
            <br>
            *.mrna: transcript sequences
            <br>
            *.cds: coding sequences
            <br>
            *.pep: protein sequences
          </div>
        </el-aside>
      </el-container>
    </el-container>
  </div>
</template>
