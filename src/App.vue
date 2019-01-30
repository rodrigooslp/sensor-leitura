<template>
  <el-container>
    <el-main>
      <el-card class="box-card">
        <div slot="header" class="clearfix">
          <el-row type="flex" justify="space-between" align="middle">
            <el-col>
              <span>Dados de Leitura de Sensor</span>
            </el-col>
            <el-col>
              <el-row type="flex" justify="end">
                <el-button type="primary" icon="el-icon-refresh" @click="loadData"></el-button>
                <el-button type="primary" @click="modalVisivel = true">Nova Leitura</el-button>
              </el-row>
            </el-col>
          </el-row>
        </div>
        <div v-loading="loading">
          <slot v-if="coleta.exists">
            <p>A coleta mais recente foi realizada em {{ coleta.dataColeta }}</p>
            <p>O índice pluviométrico foi de {{ coleta.indicePluviometrico }}.</p>
            <p>A umidade do solo foi de {{ coleta.umidadeSolo }}.</p>
          </slot>
          <slot v-else>
            <p>Nenhuma coleta foi realizada.</p>
          </slot>
        </div>
      </el-card>
    </el-main>
    <el-dialog title="Nova Leitura" :visible.sync="modalVisivel" width="60%">
      <el-form :model="form" label-position="top" ref="form" :rules="rules">
        <el-form-item label="Data de Coleta" prop="dataColeta">
          <el-date-picker v-model="form.dataColeta" type="datetime" format="dd/MM/yyyy HH:mm:ss"></el-date-picker>
        </el-form-item>
        <el-form-item label="Índice Pluviométrico" prop="indicePluviometrico">
          <el-input v-model="form.indicePluviometrico" autocomplete="off" type="number"></el-input>
        </el-form-item>
        <el-form-item label="Umidade do Solo" prop="umidadeSolo">
          <el-input v-model="form.umidadeSolo" autocomplete="off" type="number"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="modalVisivel = false">Cancelar</el-button>
        <el-button type="primary" @click="save">Salvar</el-button>
      </span>
    </el-dialog>
  </el-container>
</template>

<script>
import moment from 'moment'
import axios from 'axios'

const API_URL = process.env.API_URL
console.log(process.env)

export default {
  name: 'App',
  created () {
    this.loadData()
  },
  methods: {
    loadData () {
      this.loading = true

      const config = {
        method: 'get',
        url: 'leitura/recente',
        baseURL: API_URL
      }

      axios(config).then(({ data }) => {
        this.coleta.exists = true
        this.coleta.dataColeta = moment(data.dataColeta).format('DD/MM/YY HH:mm:ss')
        this.coleta.umidadeSolo = data.umidadeSolo
        this.coleta.indicePluviometrico = data.indicePluviometrico
        this.loading = false
      })
    },
    clearForm () {
      this.$refs.form.resetFields()
    },
    save () {
      this.$refs.form.validate((valid) => {
        if (valid) {
          const config = {
            method: 'post',
            url: 'leitura',
            baseURL: API_URL,
            data: {
              ...this.form,
              dataColeta: this.form.dataColeta.toISOString()
            }
          }
          axios(config).then(() => {
            this.modalVisivel = false
            this.$nextTick(() => {
              this.clearForm()
              this.loadData()
            })
          })
        }
      })
    }
  },
  data () {
    return {
      loading: false,
      modalVisivel: false,
      coleta: {
        exists: false,
        dataColeta: '',
        indicePluviometrico: '',
        umidadeSolo: ''
      },
      form: {
        indicePluviometrico: '',
        umidadeSolo: '',
        dataColeta: ''
      },

      rules: {
          dataColeta: [
            { required: true, message: 'A Data de Coleta é obrigatória', trigger: 'blur' },
          ],
          indicePluviometrico: [
            { required: true, message: 'O Índice Pluviométrico é obrigatório', trigger: 'blur' },
          ],
          umidadeSolo: [
            { required: true, message: 'A Umidade do Solo é obrigatória', trigger: 'blur' },
          ],
        }
    }
  }
}
</script>

<style lang="scss">
html {
  font-family: 'Roboto', sans-serif;
  box-sizing: border-box;
}

*, *:before, *:after {
    box-sizing: inherit;
}

body {
  margin: 0;
}

.el-date-editor {
  width: 100% !important;
}

.el-form-item__label {
  padding: 0 !important;
}
</style>
