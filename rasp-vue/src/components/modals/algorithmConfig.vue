<template>
  <div id="algorithmConfigModal" class="modal no-fade" tabindex="-1" role="dialog">
    <div class="modal-dialog modal-md" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">
            高级选项
          </h5>
          <button type="button" class="close" data-dismiss="modal" aria-label="Close" />
        </div>
        <div class="modal-body">

          <div v-if="key == 'sql_userinput'">
            <label class="custom-switch m-0">
              <input type="checkbox" v-model="data.pre_enable" class="custom-switch-input">
              <span class="custom-switch-indicator"></span>
              <span class="custom-switch-description">开启关键词过滤: 有漏洞但是没有攻击时不报警</span>              
            </label>

            <label class="custom-switch m-0">
              <input type="checkbox" v-model="data.allow_full" class="custom-switch-input">
              <span class="custom-switch-indicator"></span>
              <span class="custom-switch-description">允许数据库查询: 通过接口传递完整SQL语句并执行</span>              
            </label>
          </div>

          <div v-if="key == 'sql_policy'">
            <div v-for="row in sql_policy_keys" :key="row.key">
              <label class="custom-switch m-0">
                <input type="checkbox" v-model="data.feature[row.key]" class="custom-switch-input">
                <span class="custom-switch-indicator"></span>
                <span class="custom-switch-description">{{row.descr}}</span>              
              </label>
              <br>
            </div>
          </div>

          <div v-if="key == 'command_error'">
            <div v-for="row in command_error_keys" :key="row.key">
              <label class="custom-switch m-0">
                <input type="checkbox" v-model="data[row.key]" class="custom-switch-input">
                <span class="custom-switch-indicator"></span>
                <span class="custom-switch-description">{{row.descr}}</span>              
              </label>
              <br>
            </div>
          </div>

          <div v-if="key == 'xxe_disable_entity'">
            <p>将为以下类调用 setFeature() 函数，禁止外部实体加载。开启此选项可能会影响业务，请谨慎选择。</p>
            <div v-for="row in xxe_disable_entity_keys" :key="row.key">
              <label class="custom-switch m-0">
                <input type="checkbox" v-model="data[row.key]" class="custom-switch-input">
                <span class="custom-switch-indicator"></span>
                <span class="custom-switch-description">{{row.descr}}</span>              
              </label>
              <br>
            </div>
          </div>

          <div v-if="key.endsWith('_protocol')">
            <label>禁止加载的协议列表，逗号分隔</label>
            <textarea class="form-control" autocomplete="off" autocorrect="off"
              autocapitalize="off" spellcheck="false" v-model.trim="protocol_concat"></textarea>            
          </div>

          <div v-if="key == 'eval_regex'">
            <label>EVAL 语句正则表达式</label>
            <div v-bind:class="{'form-group': true, 'has-error': eval_regex_error}">
              <input type="text" v-model.trim="data.regex" class="form-control">
            </div>
            <span class="text-danger" v-if="eval_regex_error">{{eval_regex_error }}</span>
          </div>          

          <div v-if="key == 'sql_regex'">
            <label>SQL语句正则表达式 - 在编写正则时，请注意防范ReDos风险</label>
            <div v-bind:class="{'form-group': true, 'has-error': sql_regex_error}">
              <input type="text" v-model.trim="data.regex" class="form-control">
            </div>
            <span class="text-danger" v-if="sql_regex_error">{{sql_regex_error }}</span>
          </div>

          <div v-if="key == 'sql_exception'">
            <label>SQL异常代码（逗号分隔；v1.2.1 之前不支持自定义错误代码）</label>
            <div class="form-group">
              <input type="text" v-model.trim="error_code_concat" class="form-control">
            </div>
          </div>

          <div v-if="key == 'webshell_ld_preload'">
            <label>需要拦截的环境变量（一行一个）</label>
            <div class="form-group">
              <textarea type="text" style="white-space: nowrap" autocorrect="off" rows="5"
               v-model.trim="webshell_ld_preload_concat" class="form-control"></textarea>
            </div>
          </div>

          <div v-if="key == 'deserialization_blacklist'">
            <label>反序列化黑名单（一行一个）</label>
            <div class="form-group">
              <textarea type="text" style="white-space: nowrap" autocorrect="off" rows="5"
               v-model.trim="deserialization_blacklist_concat" class="form-control"></textarea>
            </div>
          </div>

          <div v-if="key == 'ognl_blacklist'">
            <label>OGNL语句黑名单（一行一个）</label>
            <div class="form-group">
              <textarea type="text" style="white-space: nowrap" autocorrect="off" rows="5"
               v-model.trim="ognl_blacklist_concat" class="form-control"></textarea>
            </div>
          </div>

          <div v-if="key == 'command_common'">
            <label>渗透命令探针 - 正则表达式</label>
            <div v-bind:class="{'form-group': true, 'has-error': command_common_error}">
              <input type="text" v-model.trim="data.pattern" class="form-control">
            </div>
            <span class="text-danger" v-if="command_common_error">{{command_common_error }}</span>
          </div>

          <div v-if="key == 'response_dataLeak'">
            <label class="custom-switch m-0">
              <input type="checkbox" v-model="data.kind.phone" class="custom-switch-input">
              <span class="custom-switch-indicator"></span>
              <span class="custom-switch-description">检测手机号泄露</span>              
            </label>
            <br/>

            <label class="custom-switch m-0">
              <input type="checkbox" v-model="data.kind.identity_card" class="custom-switch-input">
              <span class="custom-switch-indicator"></span>
              <span class="custom-switch-description">检测身份证泄露</span>              
            </label>
            <br/>

            <label class="custom-switch m-0">
              <input type="checkbox" v-model="data.kind.bank_card" class="custom-switch-input">
              <span class="custom-switch-indicator"></span>
              <span class="custom-switch-description">检测银行卡、信用卡泄露</span>              
            </label>
          </div>

        </div>
        <div class="modal-footer">
          <button class="btn btn-primary mr-auto" data-dismiss="modal" @click="saveConfig()" :disabled="sql_regex_error || command_common_error">
            确定
          </button>
          <button class="btn btn-info" data-dismiss="modal">
            取消
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters, mapActions, mapMutations } from 'vuex'
import { validateRegex, trimSplit, convertToInt } from "@/util"

export default {
  name: 'AlgorithmConfigModal',
  data: function() {
    return {
      key: '',
      data: {},
      shouldSave: false,
      command_common_error: false,
      sql_regex_error: false,
      eval_regex_error: false,
      protocol_concat: '',
      error_code_concat: '',
      deserialization_blacklist_concat: '',
      webshell_ld_preload_concat: '',
      ognl_blacklist_concat: '',
      sql_policy_keys: [
        {
          key:   'stacked_query',
          descr: '拦截多语句执行，如 select ...; update ...',
        },
        {
          key:   'no_hex',
          descr: '拦截16进制字符串，如 0x41424344',
        },
        {
          key:   'version_comment',
          descr: '拦截版本号注释，如 select/*!500001,2,*/3',
        },
        {
          key:   'function_blacklist',
          descr: '拦截黑名单函数，如 load_file、sleep、updatexml',
        },
        {
          key:   'function_count',
          descr: '函数频次算法，如 chr(123)||chr(123)||chr(123)',
        },
        {
          key:   'union_null',
          descr: '拦截连续3个NULL或者数字，如 select NULL,NULL,NULL',
        },
        {
          key:   'into_outfile',
          descr: '拦截 into outfile 写文件操作',
        },
        {
          key:   'information_schema',
          descr: '拦截 information_schema 相关操作'
        }
      ],
      command_error_keys: [
        {
          key:   'unbalanced_quote_enable',
          descr: '检查单双反引号的个数，是否为奇数'
        },
        {
          key:   'sensitive_cmd_enable',
          descr: '检查恶意的命令拼接操作，如 | bash'
        },
        {
          key:   'alarm_token_enable',
          descr: '检查恶意的 TOKEN，如 $IFS'
        }
      ],
      xxe_disable_entity_keys: [
        {
          key:   'java_dom',
          descr: 'org.apache.xerces.parsers.DOMParser'
        },
        {
          key:   'java_dom4j',
          descr: 'org.dom4j.io.SAXReader'
        },
        {
          key:   'java_jdom',
          descr: 'org.jdom.input.SAXBuilder'
        },
        {
          key:   'java_sax',
          descr: 'org.apache.xerces.internal.jaxp.SAXParserImpl'
        },
        {
          key:   'java_stax',
          descr: 'javax.xml.stream.XMLInputFactory'
        }
      ]
    }
  },
  watch: {
    'data.regex': function(newval, oldval) {
      if (this.key == 'sql_regex')
        this.sql_regex_error = this.validateRegex(newval)
    },
    // 以前的版本插件写错了，没有统一命名为 `regex`，为了兼容只能先这样了
    'data.pattern': function(newval, oldval) {
      if (this.key == 'command_common')
        this.command_common_error = this.validateRegex(newval)
    }
  },
  computed: {
    ...mapGetters(['current_app', 'app_list', 'sticky']),
  },
  mounted: function() {
    var self = this

    $('#algorithmConfigModal').on('hidden.bs.modal', function () {      
      self.setSticky(true)
    })
  },
  methods: {
    ...mapMutations(['setSticky']),
    validateRegex,
    showModal(key, data) {
      this.setSticky(false)

      this.key  = key
      this.data = JSON.parse(JSON.stringify(data))

      if (this.key.endsWith('_protocol')) {
        this.protocol_concat = this.data.protocols.join(',')
      }

      if (this.key == 'sql_exception') {
        this.error_code_concat = this.data.mysql.error_code.join(',')
      }

      if (this.key == 'webshell_ld_preload') {
        this.webshell_ld_preload_concat = this.data.env.join('\n')
      }

      if (this.key == 'deserialization_blacklist') {
        this.deserialization_blacklist_concat = this.data.clazz.join('\n')
      }

      if (this.key == 'ognl_blacklist') {
        this.ognl_blacklist_concat = this.data.expression.join('\n')
      }

      $('#algorithmConfigModal').modal({
        // backdrop: 'static',
        // keyboard: false
      })
    },
    saveConfig() {
      if (this.key == 'sql_exception') {
        this.data.mysql.error_code = convertToInt(trimSplit(this.error_code_concat, ','))
      }

      if (this.key == 'deserialization_blacklist') {
        this.data.clazz = trimSplit(this.deserialization_blacklist_concat, '\n')
      }

      if (this.key == 'ognl_blacklist') {
        this.data.expression = trimSplit(this.ognl_blacklist_concat, '\n')
      }

      if (this.key.endsWith('_protocol')) {
        this.data.protocols = trimSplit(this.protocol_concat, ',')
      }

      var body = {
        key:  this.key,
        data: this.data,
      }

      this.$emit('save', body)
    }
  }
}

</script>
