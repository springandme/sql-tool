<template>
  <h2 class="m-y-20px">SQL美化工具</h2>
  <el-form label-position="left" label-width="110px">
    <el-form-item label="数据库类型：">
      <el-select placeholder="请选择数据库类型" filterable v-model="type" class="w-220px">
        <el-option v-for="item in types" :key="item.type" :value="item.type" :label="item.name"></el-option>
      </el-select>
    </el-form-item>
    <el-form-item label="待美化SQL：">
      <el-input type="textarea" v-model="sql"
                placeholder="请输入要美化的SQL样本"
                :autosize="{ minRows: 4, maxRows: 4 }"></el-input>
    </el-form-item>
    <el-form-item label="SQL美化结果：">
      <el-input type="textarea" v-model="sqlResult" disabled
                placeholder=""
                :autosize="{ minRows: 10, maxRows: 15 }"></el-input>
    </el-form-item>
  </el-form>
  <div class="m-y-20px flex items-center justify-end">
    <el-button @click="clear">清 空</el-button>
    <el-button type="primary" @click="generateResult">生 成</el-button>
    <el-button type="primary" @click="compressSql">压缩SQL</el-button>
    <el-button @click="copyResult">复制结果</el-button>
  </div>

  <div>
    <span style="color: #999">数据库类型说明：</span>
    <el-link type="primary" v-for="item in types" v-show="item.link" @click="clickLink(item)">{{ item.name }}</el-link>
  </div>

</template>

<script setup lang="ts">
import {ElMessage} from 'element-plus'
import {copyText, openUrl} from '../../utils'
import {format} from 'sql-formatter';
import {useRoute} from 'vue-router';

const types = ref([
  {type: 'sql', name: '自动检测', link: ''},
  {type: 'mysql', name: 'MySQL', link: 'https://www.mysql.com/'},
  {type: 'plsql', name: 'Oracle PL/SQL', link: 'http://www.oracle.com/technetwork/database/features/plsql/index.html'},
  {type: 'bigquery', name: 'GCP BigQuery', link: 'https://cloud.google.com/bigquery'},
  {type: 'db2', name: 'IBM DB2', link: 'https://www.ibm.com/analytics/us/en/technology/db2/'},
  {type: 'hive', name: 'Apache Hive', link: 'https://hive.apache.org/'},
  {type: 'mariadb', name: 'MariaDB', link: 'https://mariadb.com/'},
  {type: 'n1ql', name: 'Couchbase N1QL', link: 'http://www.couchbase.com/n1ql'},
  {type: 'postgresql', name: 'PostgreSQL', link: 'https://www.postgresql.org/'},
  {
    type: 'redshift',
    name: 'Amazon Redshift',
    link: 'https://docs.aws.amazon.com/redshift/latest/dg/cm_chap_SQLCommandRef.html'
  },
  {
    type: 'singlestoredb',
    name: 'SingleStoreDB',
    link: 'https://docs.singlestore.com/managed-service/en/reference.html'
  },
  {type: 'snowflake', name: 'Snowflake', link: 'https://docs.snowflake.com/en/index.html'},
  {type: 'spark', name: 'Spark', link: 'https://spark.apache.org/docs/latest/api/sql/index.html'},
  {type: 'sqlite', name: 'SQLite', link: 'https://sqlite.org/index.html'},
  {type: 'transactsql', name: 'SQL Server Transact-SQL', link: 'https://docs.microsoft.com/en-us/sql/sql-server/'},
  {type: 'trino', name: 'Trino', link: 'https://trino.io/docs/current/'}])
const type = ref('sql')
const sql = ref('')
const sqlResult = ref('')


if (onMounted) {
  onMounted(() => {
    const route = useRoute();
    const payload = route.query?.payload
    if (payload) {
      sql.value = typeof payload === 'string' ? payload.toString() : JSON.stringify(payload)
      generateResult()
    }
  })
}

const generateResult = () => {
  if (!sql.value || sql.value.trim() == '') {
    ElMessage.info(`请输入要美化的SQL样本`)
    return
  }
  try {
    sqlResult.value = format(sql.value, {
      language: type.value,
      keywordCase: 'upper',
      expressionWidth: 200,
      tabWidth: 2,
      linesBetweenQueries: 1,
      denseOperators: false,
      newlineBeforeSemicolon: false,
      logicalOperatorNewline: 'before'
    })
  } catch (e) {
    sqlResult.value = ''
    if (type.value === 'sql') {
      return ElMessage.error('自动检测SQL失败，请指定正确的数据库类型')
    }
    return ElMessage.error('请输入正确数据库类型的SQL')
  }

  copyResult()
}

const compressSql = () => {
  if (!sql.value || sql.value.trim() == '') {
    ElMessage.info(`请输入要压缩的SQL样本`)
    return
  }
  try {
    // 移除多余的空白字符和换行符，将SQL压缩为一行
    const compressed = sql.value
      .replace(/\s+/g, ' ')  // 将多个空白字符替换为单个空格
      .replace(/\s*([,()])\s*/g, '$1')  // 移除逗号和括号周围的空格
      .replace(/\s*([=<>!+\-*/])\s*/g, ' $1 ')  // 在操作符周围保留空格
      .replace(/\s+/g, ' ')  // 再次清理多余空格
      .trim()
    
    sqlResult.value = compressed
  } catch (e) {
    sqlResult.value = ''
    return ElMessage.error('SQL压缩失败，请检查输入')
  }
}

const copyResult = () => {
  if (!sqlResult.value || sqlResult.value.trim() == '') {
    return ElMessage.info('请先生成SQL')
  }
  copyText(sqlResult.value)
  ElMessage.success('复制成功')
}

const clear = () => {
  sqlResult.value = ''
  sql.value = ''
}

const clickLink = (item: any) => openUrl(item['link'])

</script>

<style scoped lang="scss">
.el-link {
  margin-right: 20px;
}

.el-link .el-icon--right.el-icon {
  vertical-align: text-bottom;
}
</style>
