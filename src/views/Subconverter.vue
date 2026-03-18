<template>
  <div class="subconverter-page">
    <el-row class="page-row">
      <el-col>
        <el-card class="converter-card" shadow="never">
          <div slot="header" class="converter-header">
            <div class="header-side">
              <svg-icon class="header-icon gayhub" icon-class="github" @click="goToProject"/>
              <svg-icon class="header-icon dianbao" icon-class="telegram"
                      @click="gotoTgChannel"/>
            </div>
            <div class="header-title">
              <div class="header-title-cn">Awesome idea front</div>
              <div class="header-title-en">Elegant Academic Subscription Studio</div>
            </div>
            <div class="header-side header-side-right">
              <svg-icon class="header-icon bilibili" icon-class="bilibili"
                      @click="gotoBiliBili"/>
              <svg-icon class="header-icon youguan" icon-class="youtube" @click="gotoYouTuBe"/>
              <svg-icon class="header-icon channel" icon-class="telegram"
                      @click="gotoTgChannel"/>
            </div>
          </div>
          <div class="info-band">
            <div class="info-band-item">
              <span class="info-label">接口模式</span>
              <strong>{{ form.useDigest ? "Digest /digest" : "Classic /sub" }}</strong>
            </div>
            <div class="info-band-item">
              <span class="info-label">后端状态</span>
              <strong>{{ backendVersion || "待检测" }}</strong>
            </div>
            <div class="info-band-item">
              <span class="info-label">学术提示</span>
              <strong>订阅命名即 Alias（digest `a`）</strong>
            </div>
          </div>
          <div class="form-shell">
            <el-form :model="form" label-width="80px" label-position="left" class="primary-form">
              <el-divider content-position="left" class="section-divider">基础参数</el-divider>
              <el-form-item label="订阅链接:">
                <el-input
                    v-model="form.sourceSubUrl"
                    type="textarea"
                    rows="3"
                    placeholder="支持各种订阅链接或单节点链接，多个链接每行一个或用 | 分隔"
                />
              </el-form-item>
              <el-form-item label="生成类型:">
                <el-select v-model="form.clientType" style="width: 100%">
                  <el-option v-for="(v, k) in options.clientTypes" :key="k" :label="k" :value="v"></el-option>
                </el-select>
              </el-form-item>
              <el-form-item label="后端地址:">
                <el-select
                    v-model="form.customBackend"
                    allow-create
                    filterable
                    @change="selectChanged"
                    placeholder="可输入自己的后端"
                    style="width: 100%"
                >
                  <el-option v-for="(v, k) in options.customBackend" :key="k" :label="k" :value="v"></el-option>
                </el-select>
              </el-form-item>
              <el-form-item label="远程配置:">
                <el-select
                    v-model="form.remoteConfig"
                    allow-create
                    filterable
                    placeholder="请选择"
                    style="width: 100%"
                >
                  <el-option-group
                      v-for="group in options.remoteConfig"
                      :key="group.label"
                      :label="group.label"
                  >
                    <el-option
                        v-for="item in group.options"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value"
                    ></el-option>
                  </el-option-group>
                </el-select>
              </el-form-item>
              <el-form-item label="订阅命名:">
                <el-input
                    v-model="form.filename"
                    placeholder="默认 default.yaml；Digest 模式会同步为 alias(a)"
                />
              </el-form-item>
              <el-form-item label="更新间隔:">
                <el-input
                    v-model="form.interval"
                    placeholder="单位为天，默认 2"
                />
              </el-form-item>
              <el-form-item label-width="80px" class="remote-preset-item">
                <el-button
                    size="mini"
                    plain
                    type="warning"
                    @click="selectDialerRemoteConfig"
                >使用 Dialer 远程配置（默认）
                </el-button>
              </el-form-item>
              <el-alert
                  class="dense-alert"
                  title="Digest 默认启用：参数会打包到 q；订阅命名会同时作为 digest alias(a)。"
                  type="info"
                  :closable="false"
                  show-icon
              />
              <el-form-item label-width="0px">
                <el-collapse class="advanced-collapse">
                  <el-collapse-item>
                    <template slot="title">
                      <div class="advanced-toggle-header">
                        <span class="advanced-toggle-label">高级功能</span>
                        <span class="advanced-toggle-action">
                          <i class="el-icon-more-outline"></i>
                          点击显示/隐藏
                        </span>
                      </div>
                    </template>
                    <el-divider content-position="left">Dialer Proxy Providers</el-divider>
                    <el-alert
                        title="Dialer 仅对 Clash / ClashR 目标生效，默认使用 Dialer 配置，可按需切换 Dialer LoadBalance"
                        type="info"
                        :closable="false"
                        show-icon
                        style="margin-bottom: 12px"
                    />
                    <el-form-item label="Dialer开关:">
                      <el-switch
                          v-model="form.useDialer"
                          active-text="启用 use_dialer"
                          inactive-text="关闭"
                      />
                    </el-form-item>
                    <el-form-item label="Dialer组名:">
                      <el-input
                          v-model="form.dialerGroupName"
                          :disabled="!form.useDialer"
                          placeholder="默认 dialer"
                      />
                    </el-form-item>
                    <el-form-item label="应用节点:">
                      <el-input
                          v-model="form.applyDialerTo"
                          :disabled="!form.useDialer"
                          placeholder="可选正则，例如 awesome|Scholar"
                      />
                    </el-form-item>
                    <el-form-item label="Providers:">
                      <div class="provider-grid">
                        <div class="provider-column">
                          <div class="provider-guide-tip">
                            按顺序添加：新增空记录 -> 填写 name/url -> 再新增下一条。
                          </div>
                          <el-alert
                              title="每条 provider 至少需要 name 和 url；type 默认 http。"
                              type="info"
                              :closable="false"
                              show-icon
                              style="margin-bottom: 10px"
                          />
                          <div
                              v-for="(provider, index) in form.proxyProviderEntries"
                              :key="'provider-' + index"
                              class="provider-card"
                          >
                            <el-row :gutter="8">
                              <el-col :span="10">
                                <el-input
                                    v-model.trim="provider.name"
                                    placeholder="name，例如 fantastic-Vultr"
                                />
                              </el-col>
                              <el-col :span="6">
                                <el-input v-model.trim="provider.type" placeholder="type，默认 http"/>
                              </el-col>
                              <el-col :span="8" style="text-align: right;">
                                <el-button
                                    size="mini"
                                    type="danger"
                                    plain
                                    :disabled="form.proxyProviderEntries.length === 1"
                                    @click="removeProxyProviderEntry(index)"
                                >删除
                                </el-button>
                              </el-col>
                            </el-row>
                            <el-row :gutter="8" style="margin-top: 8px;">
                              <el-col :span="24">
                                <el-input v-model.trim="provider.url" placeholder="url，例如 https://example.com/sub.yaml"/>
                              </el-col>
                            </el-row>
                            <el-row :gutter="8" style="margin-top: 8px;">
                              <el-col :span="16">
                                <el-input v-model.trim="provider.path" placeholder="可选 path，例如 ./proxy_provider/custom.yaml"/>
                              </el-col>
                              <el-col :span="8">
                                <el-input v-model.trim="provider.interval" placeholder="可选 interval，例如 3600"/>
                              </el-col>
                            </el-row>
                          </div>
                          <div class="provider-actions">
                            <el-button size="mini" type="primary" plain @click="addProxyProviderEntry">新增一条 Provider</el-button>
                            <el-button size="mini" type="success" plain @click="applyDialerProvidersSample">填入示例</el-button>
                            <el-button size="mini" @click="clearProxyProviderEntries">清空 Providers</el-button>
                          </div>
                        </div>
                        <div class="provider-column provider-json-column">
                          <div class="provider-json-title">Native JSON Input (Preview)</div>
                          <div class="provider-expression-hint" v-if="dialerProviderExpressionLoading">
                            正在预取远程 ini 并识别 Dialer Provider 表达式...
                          </div>
                          <div class="provider-expression-hint provider-expression-error" v-else-if="dialerProviderExpressionError">
                            {{ dialerProviderExpressionError }}
                          </div>
                          <div class="provider-expression-hint" v-else-if="dialerProviderExpressions.length > 0">
                            允许表达式：
                            <code class="provider-expression-code">{{ dialerProviderExpressionText }}</code>
                            ，当前匹配 {{ providerPreviewStats.matched }} / {{ providerPreviewStats.total }} 条 name。
                          </div>
                          <div class="provider-expression-hint" v-else>
                            未在当前远程配置中识别到 Dialer Provider 表达式（select-use）。
                          </div>
                          <div class="provider-json-editor" :class="{ 'is-error': providerJsonPreviewState.hasError }">
                            <pre
                                ref="providerJsonOverlay"
                                class="provider-json-overlay"
                                v-html="providerJsonPreviewState.html"
                            ></pre>
                            <textarea
                                ref="providerJsonTextarea"
                                v-model="proxyProvidersJsonInput"
                                class="provider-json-textarea"
                                spellcheck="false"
                                @scroll="syncProviderJsonOverlayScroll"
                                placeholder='[{"name":"dialer-a","type":"http","url":"https://example.com/a.yaml","interval":3600}]'
                            ></textarea>
                          </div>
                          <div class="provider-actions">
                            <el-button size="mini" type="primary" @click="verifyProxyProvidersJsonWriteback">验证并回填</el-button>
                            <el-button size="mini" plain @click="refreshProxyProvidersEditorFromEntries">刷新预览</el-button>
                          </div>
                          <div class="provider-json-tip">
                            点击“验证并回填”会进行 JSON 校验，并将结果写回左侧引导输入区。
                          </div>
                        </div>
                      </div>
                    </el-form-item>
                    <el-divider content-position="left">通用高级参数</el-divider>
                    <el-form-item label="Digest开关:">
                      <el-switch
                          v-model="form.useDigest"
                          active-text="启用 /digest（默认）"
                          inactive-text="使用 /sub"
                      />
                    </el-form-item>
                    <el-form-item label="包含节点:">
                      <el-input v-model="form.includeRemarks" placeholder="要保留的节点，支持正则"/>
                    </el-form-item>
                    <el-form-item label="排除节点:">
                      <el-input v-model="form.excludeRemarks" placeholder="要排除的节点，支持正则"/>
                    </el-form-item>
                    <el-form-item label="节点命名:">
                      <el-input v-model="form.rename" placeholder="举例：`a@b``1@2`，|符可用\转义"/>
                    </el-form-item>
                    <el-form-item label="远程设备:">
                      <el-input v-model="form.devid" placeholder="用于设置QuantumultX的远程设备ID"/>
                    </el-form-item>
                    <el-form-item class="eldiy" label-width="0px">
                      <el-row type="flex">
                        <el-col>
                          <el-checkbox v-model="form.nodeList" label="仅输出节点信息" border></el-checkbox>
                        </el-col>
                        <el-popover placement="bottom" v-model="form.extraset">
                          <el-row :gutter="10">
                            <el-col :span="12">
                              <el-checkbox v-model="form.emoji" label="Emoji"></el-checkbox>
                            </el-col>
                            <el-col :span="12">
                              <el-checkbox v-model="form.insert" label="插入默认节点"></el-checkbox>
                            </el-col>
                          </el-row>
                          <el-row :gutter="10">
                            <el-col :span="12">
                              <el-checkbox v-model="form.udp" label="启用 UDP"></el-checkbox>
                            </el-col>
                            <el-col :span="12">
                              <el-checkbox v-model="form.xudp" label="启用 XUDP"></el-checkbox>
                            </el-col>
                          </el-row>
                          <el-row :gutter="10">
                            <el-col :span="12">
                              <el-checkbox v-model="form.tfo" label="启用 TFO"></el-checkbox>
                            </el-col>
                            <el-col :span="12">
                              <el-checkbox v-model="form.sort" label="基础节点排序"></el-checkbox>
                            </el-col>
                          </el-row>
                          <el-row :gutter="10">
                            <el-col :span="12">
                              <el-checkbox v-model="form.tpl.clash.doh" label="Clash.DoH"></el-checkbox>
                            </el-col>
                            <el-col :span="12">
                              <el-checkbox v-model="form.appendType" label="插入节点类型"></el-checkbox>
                            </el-col>
                          </el-row>
                          <el-row :gutter="10">
                            <el-col :span="12">
                              <el-checkbox v-model="form.tpl.surge.doh" label="Surge.DoH"></el-checkbox>
                            </el-col>
                            <el-col :span="12">
                              <el-checkbox v-model="form.tls13" label="开启TLS_1.3"></el-checkbox>
                            </el-col>
                          </el-row>
                          <el-row :gutter="10">
                            <el-col :span="12">
                              <el-checkbox v-model="form.expand" label="展开规则全文"></el-checkbox>
                            </el-col>
                            <el-col :span="12">
                              <el-checkbox v-model="form.new_name" label="Clash新字段名"></el-checkbox>
                            </el-col>
                          </el-row>
                          <el-row :gutter="10">
                            <el-col :span="12">
                              <el-checkbox v-model="form.scv" label="跳过证书验证"></el-checkbox>
                            </el-col>
                            <el-col :span="12">
                              <el-checkbox v-model="form.fdn" label="过滤不支持节点"></el-checkbox>
                            </el-col>
                          </el-row>
                          <el-row :gutter="10">
                            <el-col :span="12">
                              <div style="margin-left: 35%">
                                <el-checkbox v-model="form.tpl.singbox.ipv6" label="Sing-Box支持IPV6"></el-checkbox>
                              </div>
                            </el-col>
                          </el-row>
                          <el-button slot="reference">更多选项</el-button>
                        </el-popover>
                      </el-row>
                    </el-form-item>
                  </el-collapse-item>
                </el-collapse>
              </el-form-item>
              <div style="margin-top: 30px"></div>
              <el-divider content-position="center">
                <el-button
                    type="zhuti"
                    @click="change">
                  <i id="rijian" class="el-icon-sunny"></i>
                  <i id="yejian" class="el-icon-moon"></i>
                </el-button>
              </el-divider>
              <el-divider content-position="left" class="section-divider">输出结果</el-divider>
              <el-form-item label="定制订阅:">
                <el-input class="copy-content" disabled v-model="customSubUrl">
                  <el-button
                      slot="append"
                      v-clipboard:copy="customSubUrl"
                      v-clipboard:success="onCopy"
                      ref="copy-btn"
                      icon="el-icon-document-copy"
                  >复制
                  </el-button>
                  <el-button
                      slot="append"
                      icon="el-icon-picture-outline"
                      @click="openCustomSubQrCode"
                      :disabled="customSubUrl.length === 0"
                  >二维码
                  </el-button>
                </el-input>
              </el-form-item>
              <el-form-item label-width="0px" style="margin-top: 40px; text-align: center">
                <el-button
                    style="width: 120px"
                    type="danger"
                    @click="makeUrl"
                    :disabled="form.sourceSubUrl.length === 0 || btnBoolean"
                >生成订阅链接
                </el-button>
              </el-form-item>
              <el-form-item label-width="0px" style="text-align: center">
                <el-button
                    style="width: 120px"
                    type="primary"
                    @click="dialogUploadConfigVisible = true"
                    icon="el-icon-upload"
                    :loading="loading2"
                >自定义配置
                </el-button>
                <el-button
                    style="width: 120px"
                    type="primary"
                    @click="dialogLoadConfigVisible = true"
                    icon="el-icon-copy-document"
                    :loading="loading3"
                >从URL解析
                </el-button>
              </el-form-item>
              <el-form-item label-width="0px" style="text-align: center">
                <el-button
                    style="width: 250px;"
                    type="success"
                    icon="el-icon-video-play"
                    @click="centerDialogVisible = true"
                >视频教程
                </el-button>
              </el-form-item>
            </el-form>
          </div>
        </el-card>
      </el-col>
    </el-row>
    <el-dialog
        title="请选择需要观看的视频教程"
        :visible.sync="centerDialogVisible"
        :show-close="false"
        width="40vh"
        top="30vh"
        center>
      <div label-width="0px" style="text-align: center">
        <el-button
            style="width: 200px;"
            type="primary"
            icon="el-icon-video-play"
            @click="gotoBasicVideo();centerDialogVisible = false"
        >基础视频教程
        </el-button>
      </div>
      <div label-width="0px" style="text-align: center;margin: 3vh 0 2vh">
        <el-button
            style="width: 200px;"
            type="danger"
            icon="el-icon-video-play"
            @click="gotoAdvancedVideo();centerDialogVisible = false"
        >进阶视频教程
        </el-button>
      </div>
      <div label-width="0px" style="text-align: center;margin: 3vh 0 2vh">
        <el-button
            style="width: 200px;"
            type="warning"
            icon="el-icon-download"
            @click="toolsDown"
        >代理工具集合
        </el-button>
      </div>
    </el-dialog>
    <el-dialog
        :visible.sync="dialogUploadConfigVisible"
        :show-close="false"
        :close-on-click-modal="false"
        :close-on-press-escape="false"
        width="80%"
    >
      <el-tabs v-model="activeName" type="card">
        <el-tab-pane label="远程配置上传" name="first">
          <el-link type="danger" :href="sampleConfig" style="margin-bottom: 15px" target="_blank" icon="el-icon-info">
            参考案例
          </el-link>
          <el-form label-position="left">
            <el-form-item prop="uploadConfig">
              <el-input
                  v-model="uploadConfig"
                  type="textarea"
                  :autosize="{ minRows: 15, maxRows: 15}"
                  maxlength="50000"
                  show-word-limit
              ></el-input>
            </el-form-item>
          </el-form>
          <div style="float: right">
            <el-button type="primary" @click="uploadConfig = ''; dialogUploadConfigVisible = false">取 消</el-button>
            <el-button
                type="primary"
                @click="confirmUploadConfig"
                :disabled="uploadConfig.length === 0"
            >确 定
            </el-button>
          </div>
        </el-tab-pane>
        <el-tab-pane label="JS排序节点" name="second">
          <el-link type="success" :href="scriptConfig" style="margin-bottom: 15px" target="_blank" icon="el-icon-info">
            参考案例
          </el-link>
          <el-form label-position="left">
            <el-form-item prop="uploadScript">
              <el-input
                  v-model="uploadScript"
                  placeholder="本功能暂停使用，如有兴趣，自行去我的GitHub参考sub-web-api项目部署！"
                  type="textarea"
                  :autosize="{ minRows: 15, maxRows: 15}"
                  maxlength="50000"
                  show-word-limit
              ></el-input>
            </el-form-item>
          </el-form>
          <div style="float: right">
            <el-button type="primary" @click="uploadScript = ''; dialogUploadConfigVisible = false">取 消</el-button>
            <el-button
                type="primary"
                @click="confirmUploadScript"
                :disabled="uploadScript.length === 0"
            >确 定
            </el-button>
          </div>
        </el-tab-pane>
        <el-tab-pane label="JS筛选节点" name="third">
          <el-link type="warning" :href="filterConfig" style="margin-bottom: 15px" target="_blank" icon="el-icon-info">
            参考案例
          </el-link>
          <el-form label-position="left">
            <el-form-item prop="uploadFilter">
              <el-input
                  v-model="uploadFilter"
                  placeholder="本功能暂停使用，如有兴趣，自行去我的GitHub参考sub-web-api项目部署！"
                  type="textarea"
                  :autosize="{ minRows: 15, maxRows: 15}"
                  maxlength="50000"
                  show-word-limit
              ></el-input>
            </el-form-item>
          </el-form>
          <div style="float: right">
            <el-button type="primary" @click="uploadFilter = ''; dialogUploadConfigVisible = false">取 消</el-button>
            <el-button
                type="primary"
                @click="confirmUploadScript"
                :disabled="uploadFilter.length === 0"
            >确 定
            </el-button>
          </div>
        </el-tab-pane>
      </el-tabs>
    </el-dialog>
    <el-dialog
        :visible.sync="dialogLoadConfigVisible"
        :show-close="false"
        :close-on-click-modal="false"
        :close-on-press-escape="false"
        width="80%"
    >
      <div slot="title">
        可以从生成的长/短链接中解析信息,填入页面中去
      </div>
      <el-form label-position="left">
        <el-form-item prop="uploadConfig">
          <el-input
              v-model="loadConfig"
              type="textarea"
              :autosize="{ minRows: 15, maxRows: 15}"
              maxlength="5000"
              show-word-limit
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="loadConfig = ''; dialogLoadConfigVisible = false">取 消</el-button>
        <el-button
            type="primary"
            @click="confirmLoadConfig"
            :disabled="loadConfig.length === 0"
        >确 定
        </el-button>
      </div>
    </el-dialog>
    <el-dialog
        title="订阅二维码"
        :visible.sync="dialogQrCodeVisible"
        width="380px"
    >
      <div style="text-align: center;">
        <img
            v-if="qrCodeImageData"
            :src="qrCodeImageData"
            alt="订阅二维码"
            style="width: 320px; height: 320px; max-width: 100%;"
        />
        <el-input
            v-if="qrCodeSourceUrl"
            style="margin-top: 12px;"
            :value="qrCodeSourceUrl"
            readonly
        />
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogQrCodeVisible = false">关 闭</el-button>
        <el-button type="primary" @click="downloadQrCodeImage" :disabled="!qrCodeImageData">下载二维码</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import QRCode from "qrcode";
import pako from "pako";

const project = process.env.VUE_APP_PROJECT
const configScriptBackend = process.env.VUE_APP_CONFIG_UPLOAD_BACKEND + '/api.php'
const remoteConfigSample = process.env.VUE_APP_SUBCONVERTER_REMOTE_CONFIG
const scriptConfigSample = process.env.VUE_APP_SCRIPT_CONFIG
const filterConfigSample = process.env.VUE_APP_FILTER_CONFIG
const defaultBackend = process.env.VUE_APP_SUBCONVERTER_DEFAULT_BACKEND
const configUploadBackend = process.env.VUE_APP_CONFIG_UPLOAD_BACKEND + '/sub.php'
const defaultRemoteConfig = "https://raw.githubusercontent.com/YaoYinYing/AnyRelay/main/config/nodnsleak.ini"
const basicVideo = process.env.VUE_APP_BASIC_VIDEO
const advancedVideo = process.env.VUE_APP_ADVANCED_VIDEO
const tgBotLink = process.env.VUE_APP_BOT_LINK
const yglink = process.env.VUE_APP_YOUTUBE_LINK
const bzlink = process.env.VUE_APP_BILIBILI_LINK
const downld = 'http://' + window.location.host + '/download.html'
const digestCompactAliasByLongKey = Object.freeze({
  target: "t",
  url: "u",
  config: "c",
  include: "i",
  exclude: "e",
  rename: "r",
  dev_id: "d",
  interval: "iv",
  proxy_providers: "p",
  ver: "v",
  dialer_group_name: "dg",
  apply_dialer_to: "da"
});
const digestCompactAliasByShortKey = Object.freeze(
  Object.entries(digestCompactAliasByLongKey).reduce((acc, [longKey, shortKey]) => {
    acc[shortKey] = longKey;
    return acc;
  }, {})
);
const digestCompactBoolSpecs = Object.freeze([
  {key: "insert", defaultValue: false},
  {key: "emoji", defaultValue: true},
  {key: "list", defaultValue: false},
  {key: "xudp", defaultValue: false},
  {key: "udp", defaultValue: false},
  {key: "tfo", defaultValue: false},
  {key: "expand", defaultValue: true},
  {key: "scv", defaultValue: false},
  {key: "fdn", defaultValue: false},
  {key: "append_type", defaultValue: false},
  {key: "tls13", defaultValue: false},
  {key: "sort", defaultValue: false},
  {key: "use_dialer", defaultValue: false},
  {key: "new_name", defaultValue: true},
  {key: "surge.doh", defaultValue: false},
  {key: "clash.doh", defaultValue: false},
  {key: "singbox.ipv6", defaultValue: false}
]);
export default {
  data() {
    return {
      backendVersion: "",
      centerDialogVisible: false,
      activeName: 'first',
      // 是否为 PC 端
      isPC: true,
      btnBoolean: false,
      options: {
        clientTypes: {
          Clash: "clash",
          "Surge4/5": "surge&ver=4",
          "Sing-Box": "singbox",
          V2Ray: "v2ray",
          Trojan: "trojan",
          ShadowsocksR: "ssr",
          "混合订阅（mixed）": "mixed",
          Surfboard: "surfboard",
          Quantumult: "quan",
          "Quantumult X": "quanx",
          Loon: "loon",
          Mellow: "mellow",
          Surge3: "surge&ver=3",
          Surge2: "surge&ver=2",
          ClashR: "clashr",
          "Shadowsocks(SIP002)": "ss",
          "Shadowsocks Android(SIP008)": "sssub",
          ShadowsocksD: "ssd",
          "自动判断客户端": "auto",
        },
        customBackend: {
          "psub-oreo": "https://psub-oreo.yaoyy.moe",
          "sub-dialer": "https://sub-dialer.yaoyy.moe",
          "psub-oreo-beijing": "https://beijing-cdn.yaoyy.moe",
          "psub-oreo-chengdu": "https://chengdu-cdn.yaoyy.moe",
          "tailscale": "http://100.91.70.43:25500",
          "zerotier": "http://192.168.194.245:25500",
          "tailscale-dialer": "http://100.91.70.43:25501",
          "zerotier-dialer": "http://192.168.194.245:25501",
          "肥羊增强型后端【vless reality+hy1+hy2】": "https://url.v1.mk",
          "肥羊备用后端【vless reality+hy1+hy2】": "https://sub.d1.mk",
          "つつ-多地防失联【负载均衡+国内优化】": "https://api.tsutsu.one",
          nameless13提供: "https://www.nameless13.com",
          subconverter作者提供: "https://sub.xeton.dev",
          "sub-web作者提供": "https://api.wcc.best",
          "sub作者&lhie1提供": "https://api.dler.io",
        },
        backendOptions: [
          {value: "https://sub-dialer.yaoyy.moe"},
          {value: "http://100.91.70.43:25501"},
          {value: "http://192.168.194.245:25501"},
          {value: "https://url.v1.mk"},
          {value: "https://sub.d1.mk"},
          {value: "https://api.tsutsu.one"},
          {value: "https://www.nameless13.com"},
          {value: "https://sub.xeton.dev"},
          {value: "https://api.wcc.best"},
          {value: "https://api.dler.io"},
        ],
        remoteConfig: [
          {
            label: "AnyRelay",
            options: [
              {
                label: "No DNS leakage",
                value: "https://raw.githubusercontent.com/YaoYinYing/AnyRelay/main/config/nodnsleak.ini"
              },
              {
                label: "Dialer",
                value: "https://raw.githubusercontent.com/YaoYinYing/AnyRelay/refs/heads/main/config/nodnsleak.dialer-non_lb.ini"
              },
              {
                label: "Dialer LoadBalance",
                value: "https://raw.githubusercontent.com/YaoYinYing/AnyRelay/refs/heads/main/config/nodnsleak.dialer.ini"
              },
              {
                label: "Dialer LB Media",
                value: "https://raw.githubusercontent.com/YaoYinYing/AnyRelay/refs/heads/main/config/nodnsleak.dialer.direct-media.ini"
              },
              {
                label: "Awesome subscribe",
                value: "https://raw.githubusercontent.com/YaoYinYing/AnyRelay/main/config/nodnsleak_awesome.ini"
              },
              {
                label: "AnyRelay-LB",
                value: "https://raw.githubusercontent.com/YaoYinYing/AnyRelay/main/config/relay.ini"
              },
              {
                label: "AnyRelay-No LB",
                value: "https://raw.githubusercontent.com/YaoYinYing/AnyRelay/main/config/relay_no_lb.ini"
              }
            ]
          },
          {
            label: "通用",
            options: [
              {
                label: "ACL4SSR_Online_Full_NoAuto",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_NoAuto.ini"
              },
              {
                label: "ACL4SSR_Online_Full_AdblockPlus",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_AdblockPlus.ini"
              },
              {
                label: "默认（索尼电视专用）",
                value: "https://raw.githubusercontent.com/youshandefeiyang/webcdn/main/SONY.ini"
              },
              {
                label: "默认（附带用于 Clash 的 AdGuard DNS）",
                value: "https://gist.githubusercontent.com/tindy2013/1fa08640a9088ac8652dbd40c5d2715b/raw/default_with_clash_adg.yml"
              },
              {
                label: "ACL_全分组 Dream修改版",
                value: "https://raw.githubusercontent.com/WC-Dream/ACL4SSR/WD/Clash/config/ACL4SSR_Online_Full_Dream.ini"
              },
              {
                label: "ACL_精简分组 Dream修改版",
                value: "https://raw.githubusercontent.com/WC-Dream/ACL4SSR/WD/Clash/config/ACL4SSR_Mini_Dream.ini"
              },
              {
                label: "emby-TikTok-流媒体分组-去广告加强版",
                value: "https://raw.githubusercontent.com/justdoiting/ClashRule/main/GeneralClashRule.ini"
              },
              {
                label: "流媒体通用分组",
                value: "https://raw.githubusercontent.com/cutethotw/ClashRule/main/GeneralClashRule.ini"
              }
            ]
          },
          {
            label: "ACL规则",
            options: [
              {
                label: "ACL_默认版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online.ini"
              },
              {
                label: "ACL_无测速版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_NoAuto.ini"
              },
              {
                label: "ACL_去广告版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_AdblockPlus.ini"
              },
              {
                label: "ACL_多国家版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_MultiCountry.ini"
              },
              {
                label: "ACL_无Reject版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_NoReject.ini"
              },
              {
                label: "ACL_无测速精简版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_NoAuto.ini"
              },
              {
                label: "ACL_全分组版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full.ini"
              },
              {
                label: "ACL_全分组谷歌版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_Google.ini"
              },
              {
                label: "ACL_全分组多模式版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_MultiMode.ini"
              },
              {
                label: "ACL_全分组奈飞版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_Netflix.ini"
              },
              {
                label: "ACL_精简版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini.ini"
              },
              {
                label: "ACL_去广告精简版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_AdblockPlus.ini"
              },
              {
                label: "ACL_Fallback精简版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_Fallback.ini"
              },
              {
                label: "ACL_多国家精简版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_MultiCountry.ini"
              },
              {
                label: "ACL_多模式精简版",
                value: "https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_MultiMode.ini"
              }
            ]
          },
          {
            label: "全网搜集规则",
            options: [
              {
                label: "常规规则",
                value: "https://raw.githubusercontent.com/flyhigherpi/merlinclash_clash_related/master/Rule_config/ZHANG.ini"
              },
              {
                label: "酷酷自用",
                value: "https://raw.githubusercontent.com/xiaoshenxian233/cool/rule/complex.ini"
              },
              {
                label: "PharosPro无测速",
                value:
                    "https://subweb.s3.fr-par.scw.cloud/RemoteConfig/special/phaors.ini"
              },
              {
                label: "分区域故障转移",
                value: "https://raw.githubusercontent.com/flyhigherpi/merlinclash_clash_related/master/Rule_config/ZHANG_Area_Fallback.ini"
              },
              {
                label: "分区域自动测速",
                value: "https://raw.githubusercontent.com/flyhigherpi/merlinclash_clash_related/master/Rule_config/ZHANG_Area_Urltest.ini"
              },
              {
                label: "分区域无自动测速",
                value: "https://raw.githubusercontent.com/flyhigherpi/merlinclash_clash_related/master/Rule_config/ZHANG_Area_NoAuto.ini"
              },
              {
                label: "OoHHHHHHH",
                value: "https://raw.githubusercontent.com/OoHHHHHHH/ini/master/config.ini"
              },
              {
                label: "CFW-TAP",
                value: "https://raw.githubusercontent.com/OoHHHHHHH/ini/master/cfw-tap.ini"
              },
              {
                label: "lhl77全分组（定期更新）",
                value: "https://raw.githubusercontent.com/lhl77/sub-ini/main/tsutsu-full.ini"
              },
              {
                label: "lhl77简易版（定期更新）",
                value: "https://raw.githubusercontent.com/lhl77/sub-ini/main/tsutsu-mini-gfw.ini"
              },
              {
                label: "ConnersHua 神机规则 Outbound",
                value: "https://gist.githubusercontent.com/tindy2013/1fa08640a9088ac8652dbd40c5d2715b/raw/connershua_new.ini"
              },
              {
                label: "ConnersHua 神机规则 Inbound 回国专用",
                value: "https://gist.githubusercontent.com/tindy2013/1fa08640a9088ac8652dbd40c5d2715b/raw/connershua_backtocn.ini"
              },
              {
                label: "lhie1 洞主规则（使用 Clash 分组规则）",
                value: "https://gist.githubusercontent.com/tindy2013/1fa08640a9088ac8652dbd40c5d2715b/raw/lhie1_clash.ini"
              },
              {
                label: "lhie1 洞主规则完整版",
                value: "https://gist.githubusercontent.com/tindy2013/1fa08640a9088ac8652dbd40c5d2715b/raw/lhie1_dler.ini"
              },
              {
                label: "eHpo1 规则",
                value: "https://gist.githubusercontent.com/tindy2013/1fa08640a9088ac8652dbd40c5d2715b/raw/ehpo1_main.ini"
              },
              {
                label: "多策略组默认白名单模式",
                value: "https://raw.nameless13.com/api/public/dl/ROzQqi2S/white.ini"
              },
              {
                label: "多策略组可以有效减少审计触发",
                value: "https://raw.nameless13.com/api/public/dl/ptLeiO3S/mayinggfw.ini"
              },
              {
                label: "精简策略默认白名单",
                value: "https://raw.nameless13.com/api/public/dl/FWSh3dXz/easy3.ini"
              },
              {
                label: "多策略增加SMTP策略",
                value: "https://raw.nameless13.com/api/public/dl/L_-vxO7I/youtube.ini"
              },
              {
                label: "无策略入门推荐",
                value: "https://raw.nameless13.com/api/public/dl/zKF9vFbb/easy.ini"
              },
              {
                label: "无策略入门推荐国家分组",
                value: "https://raw.nameless13.com/api/public/dl/E69bzCaE/easy2.ini"
              },
              {
                label: "无策略仅IPIP CN + Final",
                value: "https://raw.nameless13.com/api/public/dl/XHr0miMg/ipip.ini"
              },
              {
                label: "无策略魅影vip分组",
                value: "https://raw.nameless13.com/api/public/dl/BBnfb5lD/MAYINGVIP.ini"
              },
              {
                label: "品云专属配置（仅香港区域分组）",
                value: "https://raw.githubusercontent.com/Mazeorz/airports/master/Clash/Examine.ini"
              },
              {
                label: "品云专属配置（全地域分组）",
                value: "https://raw.githubusercontent.com/Mazeorz/airports/master/Clash/Examine_Full.ini"
              },
              {
                label: "nzw9314 规则",
                value: "https://gist.githubusercontent.com/tindy2013/1fa08640a9088ac8652dbd40c5d2715b/raw/nzw9314_custom.ini"
              },
              {
                label: "maicoo-l 规则",
                value: "https://gist.githubusercontent.com/tindy2013/1fa08640a9088ac8652dbd40c5d2715b/raw/maicoo-l_custom.ini"
              },
              {
                label: "DlerCloud Platinum 李哥定制规则",
                value: "https://gist.githubusercontent.com/tindy2013/1fa08640a9088ac8652dbd40c5d2715b/raw/dlercloud_lige_platinum.ini"
              },
              {
                label: "DlerCloud Gold 李哥定制规则",
                value: "https://gist.githubusercontent.com/tindy2013/1fa08640a9088ac8652dbd40c5d2715b/raw/dlercloud_lige_gold.ini"
              },
              {
                label: "DlerCloud Silver 李哥定制规则",
                value: "https://gist.githubusercontent.com/tindy2013/1fa08640a9088ac8652dbd40c5d2715b/raw/dlercloud_lige_silver.ini"
              },
              {
                label: "ProxyStorage自用",
                value: "https://unpkg.com/proxy-script/config/Clash/clash.ini"
              },
              {
                label: "ShellClash修改版规则 (by UlinoyaPed)",
                value: "https://github.com/UlinoyaPed/ShellClash/raw/master/rules/ShellClash.ini"
              }
            ]
          },
          {
            label: "各大机场规则",
            options: [
              {
                label: "EXFLUX",
                value:
                    "https://gist.github.com/jklolixxs/16964c46bad1821c70fa97109fd6faa2/raw/EXFLUX.ini"
              },
              {
                label: "NaNoport",
                value:
                    "https://gist.github.com/jklolixxs/32d4e9a1a5d18a92beccf3be434f7966/raw/NaNoport.ini"
              },
              {
                label: "CordCloud",
                value:
                    "https://gist.github.com/jklolixxs/dfbe0cf71ffc547557395c772836d9a8/raw/CordCloud.ini"
              },
              {
                label: "BigAirport",
                value:
                    "https://gist.github.com/jklolixxs/e2b0105c8be6023f3941816509a4c453/raw/BigAirport.ini"
              },
              {
                label: "跑路云",
                value:
                    "https://gist.github.com/jklolixxs/9f6989137a2cfcc138c6da4bd4e4cbfc/raw/PaoLuCloud.ini"
              },
              {
                label: "WaveCloud",
                value:
                    "https://gist.github.com/jklolixxs/fccb74b6c0018b3ad7b9ed6d327035b3/raw/WaveCloud.ini"
              },
              {
                label: "几鸡",
                value:
                    "https://gist.github.com/jklolixxs/bfd5061dceeef85e84401482f5c92e42/raw/JiJi.ini"
              },
              {
                label: "四季加速",
                value:
                    "https://gist.github.com/jklolixxs/6ff6e7658033e9b535e24ade072cf374/raw/SJ.ini"
              },
              {
                label: "ImmTelecom",
                value:
                    "https://gist.github.com/jklolixxs/24f4f58bb646ee2c625803eb916fe36d/raw/ImmTelecom.ini"
              },
              {
                label: "AmyTelecom",
                value:
                    "https://gist.github.com/jklolixxs/b53d315cd1cede23af83322c26ce34ec/raw/AmyTelecom.ini"
              },
              {
                label: "LinkCube",
                value:
                    "https://subweb.s3.fr-par.scw.cloud/RemoteConfig/customized/convenience.ini"
              },
              {
                label: "Miaona",
                value:
                    "https://gist.github.com/jklolixxs/ff8ddbf2526cafa568d064006a7008e7/raw/Miaona.ini"
              },
              {
                label: "Foo&Friends",
                value:
                    "https://gist.github.com/jklolixxs/df8fda1aa225db44e70c8ac0978a3da4/raw/Foo&Friends.ini"
              },
              {
                label: "ABCloud",
                value:
                    "https://gist.github.com/jklolixxs/b1f91606165b1df82e5481b08fd02e00/raw/ABCloud.ini"
              },
              {
                label: "咸鱼",
                value: "https://raw.githubusercontent.com/SleepyHeeead/subconverter-config/master/remote-config/customized/xianyu.ini"
              },
              {
                label: "便利店",
                value: "https://subweb.oss-cn-hongkong.aliyuncs.com/RemoteConfig/customized/convenience.ini"
              },
              {
                label: "CNIX",
                value: "https://raw.githubusercontent.com/Mazeorz/airports/master/Clash/SSRcloud.ini"
              },
              {
                label: "Nirvana",
                value: "https://raw.githubusercontent.com/Mazetsz/ACL4SSR/master/Clash/config/V2rayPro.ini"
              },
              {
                label: "V2Pro",
                value: "https://raw.githubusercontent.com/Mazeorz/airports/master/Clash/V2Pro.ini"
              },
              {
                label: "史迪仔-自动测速",
                value: "https://raw.githubusercontent.com/Mazeorz/airports/master/Clash/Stitch.ini"
              },
              {
                label: "史迪仔-负载均衡",
                value: "https://raw.githubusercontent.com/Mazeorz/airports/master/Clash/Stitch-Balance.ini"
              },
              {
                label: "Maying",
                value: "https://raw.githubusercontent.com/SleepyHeeead/subconverter-config/master/remote-config/customized/maying.ini"
              },
              {
                label: "Ytoo",
                value: "https://subweb.s3.fr-par.scw.cloud/RemoteConfig/customized/ytoo.ini"
              },
              {
                label: "w8ves",
                value: "https://raw.nameless13.com/api/public/dl/M-We_Fn7/w8ves.ini"
              },
              {
                label: "NyanCAT",
                value: "https://raw.githubusercontent.com/SleepyHeeead/subconverter-config/master/remote-config/customized/nyancat.ini"
              },
              {
                label: "Nexitally",
                value: "https://subweb.s3.fr-par.scw.cloud/RemoteConfig/customized/nexitally.ini"
              },
              {
                label: "SoCloud",
                value: "https://raw.githubusercontent.com/SleepyHeeead/subconverter-config/master/remote-config/customized/socloud.ini"
              },
              {
                label: "ARK",
                value: "https://raw.githubusercontent.com/SleepyHeeead/subconverter-config/master/remote-config/customized/ark.ini"
              },
              {
                label: "N3RO",
                value: "https://gist.githubusercontent.com/tindy2013/1fa08640a9088ac8652dbd40c5d2715b/raw/n3ro_optimized.ini"
              },
              {
                label: "Scholar",
                value: "https://gist.githubusercontent.com/tindy2013/1fa08640a9088ac8652dbd40c5d2715b/raw/scholar_optimized.ini"
              },
              {
                label: "Flowercloud",
                value: "https://subweb.s3.fr-par.scw.cloud/RemoteConfig/customized/flower.ini"
              }
            ]
          },
          {
            label: "特殊",
            options: [
              {
                label: "NeteaseUnblock",
                value: "https://raw.githubusercontent.com/SleepyHeeead/subconverter-config/master/remote-config/special/netease.ini"
              },
              {
                label: "Basic",
                value: "https://raw.githubusercontent.com/SleepyHeeead/subconverter-config/master/remote-config/special/basic.ini"
              }
            ]
          }
        ]
      },
      form: {
        sourceSubUrl: "",
        clientType: "",
        customBackend: this.getUrlParam() == "" ? "https://psub-oreo.yaoyy.moe" : this.getUrlParam(),
        remoteConfig: defaultRemoteConfig,
        excludeRemarks: "",
        includeRemarks: "",
        filename: "default.yaml",
        rename: "",
        devid: "",
        interval: "2",
        emoji: true,
        nodeList: false,
        extraset: false,
        useDigest: true,
        tls13: false,
        udp: false,
        xudp: false,
        tfo: false,
        sort: false,
        expand: true,
        scv: false,
        fdn: false,
        appendType: false,
        insert: false, // 是否插入默认订阅的节点，对应配置项 insert_url
        new_name: true, // 是否使用 Clash 新字段
        useDialer: false,
        dialerGroupName: "dialer",
        applyDialerTo: "",
        proxyProviders: "",
        proxyProviderEntries: [
          {
            name: "",
            type: "http",
            url: "",
            path: "",
            interval: ""
          }
        ],
        tpl: {
          surge: {
            doh: false // dns 查询是否使用 DoH
          },
          clash: {
            doh: false
          },
          singbox: {
            ipv6: false
          }
        }
      },
      loading2: false,
      loading3: false,
      customSubUrl: "",
      dialogUploadConfigVisible: false,
      loadConfig: "",
      dialogLoadConfigVisible: false,
      dialogQrCodeVisible: false,
      qrCodeImageData: "",
      qrCodeSourceUrl: "",
      uploadFilter: "",
      uploadScript: "",
      uploadConfig: "",
      myBot: tgBotLink,
      filterConfig: filterConfigSample,
      scriptConfig: scriptConfigSample,
      sampleConfig: remoteConfigSample,
      proxyProvidersJsonInput: "",
      dialerProviderExpressions: [],
      dialerProviderRegexList: [],
      dialerProviderExpressionLoading: false,
      dialerProviderExpressionError: "",
      remoteConfigExpressionCache: {},
      remoteConfigPrefetchTimer: null
    };
  },
  created() {
    document.title = "Awesome idea front";
    this.isPC = this.$getOS().isPc;
  },
  beforeDestroy() {
    if (this.remoteConfigPrefetchTimer) {
      clearTimeout(this.remoteConfigPrefetchTimer);
    }
  },
  mounted() {
    // this.tanchuang();
    this.form.clientType = "clash";
    this.refreshProxyProvidersEditorFromEntries();
    this.getBackendVersion();
    this.anhei();
    let lightMedia = window.matchMedia('(prefers-color-scheme: light)');
    let darkMedia = window.matchMedia('(prefers-color-scheme: dark)');
    let callback = (e) => {
      if (e.matches) {
        this.anhei();
      }
    };
    if (typeof darkMedia.addEventListener === 'function' || typeof lightMedia.addEventListener === 'function') {
      lightMedia.addEventListener('change', callback);
      darkMedia.addEventListener('change', callback);
    } //监听系统主题，自动切换！
  },
  computed: {
    dialerProviderExpressionText() {
      return this.dialerProviderExpressions.join(" | ");
    },
    providerJsonPreviewState() {
      return this.buildProviderJsonPreviewState();
    },
    providerPreviewStats() {
      return {
        total: this.providerJsonPreviewState.total,
        matched: this.providerJsonPreviewState.matched
      };
    }
  },
  watch: {
    "form.proxyProviderEntries": {
      deep: true,
      handler() {
        this.syncProxyProvidersStringFromEntries();
        this.refreshProxyProvidersEditorFromEntries();
      }
    },
    "form.remoteConfig": {
      immediate: true,
      handler() {
        this.scheduleDialerExpressionPrefetch();
      }
    },
    "form.useDialer"(enabled) {
      if (enabled) {
        this.ensureDialerBackendSupported(true);
      }
    },
    "form.customBackend"() {
      if (this.form.useDialer) {
        this.ensureDialerBackendSupported(true);
      }
    },
    proxyProvidersJsonInput() {
      this.$nextTick(() => {
        this.syncProviderJsonOverlayScroll();
      });
    }
  },
  methods: {
    notify(type, message, options = {}) {
      const extraClass = typeof options.customClass === "string" && options.customClass.trim() !== "" ? ` ${options.customClass.trim()}` : "";
      return this.$message({
        showClose: true,
        duration: type === "error" ? 3200 : 2200,
        ...options,
        type,
        message,
        customClass: `message-bottom-right${extraClass}`.trim()
      });
    },
    notifySuccess(message, options = {}) {
      return this.notify("success", message, options);
    },
    notifyError(message, options = {}) {
      return this.notify("error", message, options);
    },
    isDialerBackendSupported(backend = this.form.customBackend) {
      const text = typeof backend === "string" ? backend : "";
      return text.includes("sub-dialer.yaoyy.moe");
    },
    ensureDialerBackendSupported(needNotify = false) {
      if (!this.form.useDialer) {
        return true;
      }
      if (this.isDialerBackendSupported()) {
        return true;
      }
      if (needNotify) {
        this.notifyError("该后端不支持dialer");
      }
      return false;
    },
    selectChanged() {
      this.getBackendVersion();
    },
    getUrlParam() {
      let query = window.location.search.substring(1);
      let vars = query.split('&');
      for (let i = 0; i < vars.length; i++) {
        var pair = vars[i].split('=');
        if (pair[0] == "backend") {
          return decodeURIComponent(pair[1]);
        }
      }
      return "";
    },
    anhei() {
      const getLocalTheme = window.localStorage.getItem("localTheme");
      const lightMode = window.matchMedia && window.matchMedia('(prefers-color-scheme: light)');
      const darkMode = window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)');
      if (getLocalTheme) {
        document.getElementsByTagName('body')[0].className = getLocalTheme;
      } //读取localstorage，优先级最高！
      else if (getLocalTheme == null || getLocalTheme == "undefined" || getLocalTheme == "") {
        if (new Date().getHours() >= 19 || new Date().getHours() < 7) {
          document.getElementsByTagName('body')[0].setAttribute('class', 'dark-mode');
        } else {
          document.getElementsByTagName('body')[0].setAttribute('class', 'light-mode');
        } //根据当前时间来判断，用来对付QQ等不支持媒体变量查询的浏览器
        if (lightMode && lightMode.matches) {
          document.getElementsByTagName('body')[0].setAttribute('class', 'light-mode');
        }
        if (darkMode && darkMode.matches) {
          document.getElementsByTagName('body')[0].setAttribute('class', 'dark-mode');
        } //根据窗口主题来判断当前主题！
      }
    },
    change() {
      var zhuti = document.getElementsByTagName('body')[0].className;
      if (zhuti === 'light-mode') {
        document.getElementsByTagName('body')[0].setAttribute('class', 'dark-mode');
        window.localStorage.setItem('localTheme', 'dark-mode');
      }
      if (zhuti === 'dark-mode') {
        document.getElementsByTagName('body')[0].setAttribute('class', 'light-mode');
        window.localStorage.setItem('localTheme', 'light-mode');
      }
    },
    // tanchuang() {
    //   this.$alert(`<div style="text-align:center;font-size:15px"><strong><span style="font-size:20px;color:red">apiurl.v1.mk已被蔷，请更换最新的url.v1.mk</span></strong></br><strong><span style="font-size:20px">本站官方TG交流群：</span><span><a href="https://t.me/feiyangdigital" target="_blank" style="color:red;font-size:20px;text-decoration:none">点击加入</a></span></strong></br><strong><span style="font-size:20px">IEPL高端机场（<span style="color:blue">原生支持奈飞非自制剧、Disney Plus、HBO等各种流媒体，支持Chat-GPT和ISP住宅IP助力Tiktok等跨境贸易使用</span>）：</span><span><a href="https://www.mcwy.org" style="color:red;font-size:20px;text-decoration:none">点击注册</a></span></strong></br><strong><span style="font-size:20px">奈飞、ChatGPT合租（<span style="color:blue">优惠码：feiyang</span>）：</span><span><a href="https://hezu.v1.mk/" style="color:red;font-size:20px;text-decoration:none">点击上车</a></span></strong></br><strong><span style="font-size:20px">115蓝光4K原盘内部资源群：</span><span><a href="https://115.metshop.top" target="_blank" style="color:red;font-size:20px;text-decoration:none">点击查看</a></span></strong></br>本站服务器赞助机场-牧场物语，是一家拥有BGP中继+IEPL企业级内网专线的高端机场，适合各个价位要求的用户，牧场物语采用最新的奈飞非自制剧解决方案，出口随机更换IP，确保尽可能的每个用户可以用上独立IP，以此来稳定解决奈飞非自制剧的封锁，并推出7*24小时奈飞非自制剧节点自动检测系统，用户再也不用自己手动一个个的乱试节点了，目前牧场的新加坡，台湾等节区域点均可做到24H稳定非自制剧观看，支持Chat-GPT和ISP住宅IP助力Tiktok等跨境贸易使用！</br></div>`, '信息面板', {
    //     confirmButtonText: '确定',
    //     dangerouslyUseHTMLString: true,
    //     customClass: 'msgbox'
    //   });
    // },
    onCopy() {
      this.notifySuccess("已复制");
    },
    async openQrCodeDialog(url) {
      const targetUrl = typeof url === "string" ? url.trim() : "";
      if (targetUrl === "") {
        this.notifyError("链接为空，无法生成二维码");
        return;
      }
      try {
        const qrWidth = targetUrl.length > 900 ? 460 : (targetUrl.length > 650 ? 420 : 360);
        this.qrCodeImageData = await QRCode.toDataURL(targetUrl, {
          width: qrWidth,
          margin: 1,
          errorCorrectionLevel: "L"
        });
        this.qrCodeSourceUrl = targetUrl;
        this.dialogQrCodeVisible = true;
      } catch (e) {
        this.notifyError("二维码生成失败");
      }
    },
    openCustomSubQrCode() {
      this.openQrCodeDialog(this.customSubUrl);
    },
    downloadQrCodeImage() {
      if (!this.qrCodeImageData) {
        return;
      }
      const anchor = document.createElement("a");
      anchor.href = this.qrCodeImageData;
      anchor.download = "subscription-qrcode.png";
      document.body.appendChild(anchor);
      anchor.click();
      document.body.removeChild(anchor);
    },
    goToProject() {
      window.open(project);
    },
    gotoTgChannel() {
      window.open(tgBotLink);
    },
    gotoBiliBili() {
      window.open(bzlink);
    },
    gotoYouTuBe() {
      window.open(yglink);
    },
    toolsDown() {
      window.open(downld);
    },
    gotoBasicVideo() {
      this.$alert("别忘了关注友善的肥羊哦！", {
        type: "warning",
        confirmButtonText: '确定',
        customClass: 'msgbox',
        showClose: false,
      })
          .then(() => {
            window.open(basicVideo);
          });
    },
    gotoAdvancedVideo() {
      this.$alert("别忘了关注友善的肥羊哦！", {
        type: "warning",
        confirmButtonText: '确定',
        customClass: 'msgbox',
        showClose: false,
      })
          .then(() => {
            window.open(advancedVideo);
          });
    },
    selectDialerRemoteConfig() {
      this.form.remoteConfig = "https://raw.githubusercontent.com/YaoYinYing/AnyRelay/refs/heads/main/config/nodnsleak.dialer-non_lb.ini";
      this.notifySuccess("已切换到 Dialer（默认）远程配置");
    },
    scheduleDialerExpressionPrefetch() {
      if (this.remoteConfigPrefetchTimer) {
        clearTimeout(this.remoteConfigPrefetchTimer);
      }
      this.remoteConfigPrefetchTimer = setTimeout(() => {
        this.prefetchDialerExpressionFromRemoteConfig();
      }, 300);
    },
    async prefetchDialerExpressionFromRemoteConfig() {
      const remoteConfigUrl = typeof this.form.remoteConfig === "string" ? this.form.remoteConfig.trim() : "";
      this.dialerProviderExpressionError = "";
      if (remoteConfigUrl === "" || !/^https?:\/\//i.test(remoteConfigUrl)) {
        this.dialerProviderExpressions = [];
        this.dialerProviderRegexList = [];
        return;
      }
      if (Object.prototype.hasOwnProperty.call(this.remoteConfigExpressionCache, remoteConfigUrl)) {
        const cached = this.remoteConfigExpressionCache[remoteConfigUrl];
        this.dialerProviderExpressions = cached.expressions;
        this.dialerProviderRegexList = cached.regexList;
        return;
      }
      this.dialerProviderExpressionLoading = true;
      try {
        const response = await fetch(remoteConfigUrl, {method: "GET"});
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`);
        }
        const iniText = await response.text();
        const expressions = this.extractDialerProviderExpressionsFromIni(iniText);
        const regexList = this.compileDialerProviderExpressions(expressions);
        this.remoteConfigExpressionCache[remoteConfigUrl] = {expressions, regexList};
        if (this.form.remoteConfig.trim() === remoteConfigUrl) {
          this.dialerProviderExpressions = expressions;
          this.dialerProviderRegexList = regexList;
        }
      } catch (error) {
        if (this.form.remoteConfig.trim() === remoteConfigUrl) {
          this.dialerProviderExpressions = [];
          this.dialerProviderRegexList = [];
          this.dialerProviderExpressionError = "预取 ini 失败，当前无法进行表达式匹配高亮";
        }
      } finally {
        if (this.form.remoteConfig.trim() === remoteConfigUrl) {
          this.dialerProviderExpressionLoading = false;
        }
      }
    },
    extractDialerProviderExpressionsFromIni(iniText) {
      if (typeof iniText !== "string" || iniText.trim() === "") {
        return [];
      }
      const expressionSet = new Set();
      iniText.split(/\r?\n/).forEach(line => {
        const content = line.trim();
        if (content === "" || content.startsWith("#") || content.startsWith(";")) {
          return;
        }
        if (!content.startsWith("custom_proxy_group=")) {
          return;
        }
        const raw = content.slice("custom_proxy_group=".length);
        const tokens = raw.split("`");
        if (tokens.length < 3) {
          return;
        }
        const groupName = tokens[0].trim();
        const groupType = tokens[1].trim().toLowerCase();
        const expression = tokens[2].trim();
        if (groupType !== "select-use") {
          return;
        }
        if (!/dialer/i.test(groupName)) {
          return;
        }
        if (expression !== "") {
          expressionSet.add(expression);
        }
      });
      return Array.from(expressionSet);
    },
    compileDialerProviderExpressions(expressions) {
      if (!Array.isArray(expressions)) {
        return [];
      }
      const regexList = [];
      expressions.forEach(expression => {
        if (typeof expression !== "string") {
          return;
        }
        let pattern = expression.trim();
        if (pattern === "") {
          return;
        }
        if ((pattern.startsWith("'") && pattern.endsWith("'")) || (pattern.startsWith("\"") && pattern.endsWith("\""))) {
          pattern = pattern.slice(1, -1);
        }
        let flags = "";
        const inlineFlags = pattern.match(/^\(\?([A-Za-z]+)\)/);
        if (inlineFlags) {
          flags = inlineFlags[1].toLowerCase().split("").filter(ch => "imsuy".includes(ch)).join("");
          pattern = pattern.slice(inlineFlags[0].length);
        }
        if (pattern === "") {
          return;
        }
        try {
          regexList.push(new RegExp(pattern, flags));
        } catch (e) {
          // ignore invalid expression in ini
        }
      });
      return regexList;
    },
    escapeHtml(content) {
      return String(content)
          .replace(/&/g, "&amp;")
          .replace(/</g, "&lt;")
          .replace(/>/g, "&gt;")
          .replace(/"/g, "&quot;")
          .replace(/'/g, "&#39;");
    },
    mergeRanges(ranges) {
      if (!Array.isArray(ranges) || ranges.length === 0) {
        return [];
      }
      const sorted = ranges
          .filter(item => Array.isArray(item) && item.length === 2 && item[1] > item[0])
          .sort((a, b) => a[0] - b[0]);
      if (sorted.length === 0) {
        return [];
      }
      const merged = [sorted[0].slice()];
      for (let i = 1; i < sorted.length; i++) {
        const current = sorted[i];
        const last = merged[merged.length - 1];
        if (current[0] <= last[1]) {
          last[1] = Math.max(last[1], current[1]);
        } else {
          merged.push(current.slice());
        }
      }
      return merged;
    },
    findMatchedRangesInProviderName(name) {
      if (typeof name !== "string" || name === "" || this.dialerProviderRegexList.length === 0) {
        return [];
      }
      const ranges = [];
      this.dialerProviderRegexList.forEach(regex => {
        const scanFlags = Array.from(new Set((regex.flags + "g").split(""))).join("");
        const scanRegex = new RegExp(regex.source, scanFlags);
        let count = 0;
        let match = scanRegex.exec(name);
        while (match) {
          if (match[0] !== "") {
            ranges.push([match.index, match.index + match[0].length]);
          } else {
            scanRegex.lastIndex += 1;
          }
          count += 1;
          if (count > 200) {
            break;
          }
          match = scanRegex.exec(name);
        }
      });
      return this.mergeRanges(ranges);
    },
    renderHighlightedProviderName(name) {
      const ranges = this.findMatchedRangesInProviderName(name);
      if (ranges.length === 0) {
        return this.escapeHtml(name);
      }
      let cursor = 0;
      let html = "";
      ranges.forEach(([start, end]) => {
        if (cursor < start) {
          html += this.escapeHtml(name.slice(cursor, start));
        }
        html += `<mark class="provider-name-match">${this.escapeHtml(name.slice(start, end))}</mark>`;
        cursor = end;
      });
      if (cursor < name.length) {
        html += this.escapeHtml(name.slice(cursor));
      }
      return html;
    },
    syncProviderJsonOverlayScroll() {
      const textarea = this.$refs.providerJsonTextarea;
      const overlay = this.$refs.providerJsonOverlay;
      if (!textarea || !overlay) {
        return;
      }
      overlay.scrollTop = textarea.scrollTop;
      overlay.scrollLeft = textarea.scrollLeft;
    },
    buildProviderJsonPreviewState() {
      const raw = typeof this.proxyProvidersJsonInput === "string" ? this.proxyProvidersJsonInput.trim() : "";
      if (raw === "") {
        return {
          html: "&nbsp;",
          total: 0,
          matched: 0,
          hasError: false
        };
      }
      let parsed;
      try {
        parsed = JSON.parse(raw);
      } catch (error) {
        return {
          html: this.escapeHtml(this.proxyProvidersJsonInput),
          total: 0,
          matched: 0,
          hasError: true
        };
      }
      let total = 0;
      let matched = 0;
      const nameTokens = [];
      const tokenize = (value) => {
        if (Array.isArray(value)) {
          return value.map(item => tokenize(item));
        }
        if (!value || typeof value !== "object") {
          return value;
        }
        const result = {};
        Object.keys(value).forEach(key => {
          const fieldValue = value[key];
          if (key === "name" && typeof fieldValue === "string") {
            const highlighted = this.renderHighlightedProviderName(fieldValue);
            total += 1;
            if (highlighted !== this.escapeHtml(fieldValue)) {
              matched += 1;
            }
            const token = `__DIALER_PROVIDER_TOKEN_${nameTokens.length}__`;
            nameTokens.push(highlighted);
            result[key] = token;
          } else {
            result[key] = tokenize(fieldValue);
          }
        });
        return result;
      };
      const tokenized = tokenize(parsed);
      let html = this.escapeHtml(JSON.stringify(tokenized, null, 2));
      html = html.replace(/&quot;__DIALER_PROVIDER_TOKEN_(\d+)__&quot;/g, (token, indexText) => {
        const index = parseInt(indexText, 10);
        return nameTokens[index] === undefined ? token : `&quot;${nameTokens[index]}&quot;`;
      });
      return {
        html,
        total,
        matched,
        hasError: false
      };
    },
    applyDialerProvidersSample() {
      this.form.proxyProviderEntries = [
        {name: "sub-dialer-provider-1", type: "http", url: "https://example.com/sub.yaml", path: "", interval: "3600"},
        {name: "relay-provider-1", type: "http", url: "https://example.com/relay.yaml", path: "", interval: "3600"}
      ];
      this.syncProxyProvidersStringFromEntries();
      this.refreshProxyProvidersEditorFromEntries();
    },
    createEmptyProxyProviderEntry() {
      return {
        name: "",
        type: "http",
        url: "",
        path: "",
        interval: ""
      };
    },
    addProxyProviderEntry() {
      this.form.proxyProviderEntries.push(this.createEmptyProxyProviderEntry());
    },
    removeProxyProviderEntry(index) {
      this.form.proxyProviderEntries.splice(index, 1);
      if (this.form.proxyProviderEntries.length === 0) {
        this.form.proxyProviderEntries.push(this.createEmptyProxyProviderEntry());
      }
      this.syncProxyProvidersStringFromEntries();
    },
    clearProxyProviderEntries() {
      this.form.proxyProviderEntries = [this.createEmptyProxyProviderEntry()];
      this.form.proxyProviders = "";
      this.refreshProxyProvidersEditorFromEntries();
    },
    normalizeProxyProviderEntry(item, index = 0) {
      if (!item || typeof item !== "object" || Array.isArray(item)) {
        throw new Error(`第 ${index + 1} 条 provider 必须是对象`);
      }
      const name = typeof item.name === "string" ? item.name.trim() : "";
      const url = typeof item.url === "string" ? item.url.trim() : "";
      const type = typeof item.type === "string" && item.type.trim() !== "" ? item.type.trim() : "http";
      const path = typeof item.path === "string" ? item.path.trim() : "";
      const rawInterval = item.interval === undefined || item.interval === null ? "" : String(item.interval).trim();
      let interval = "";
      if (rawInterval !== "") {
        const parsedInterval = parseInt(rawInterval, 10);
        if (Number.isNaN(parsedInterval) || parsedInterval <= 0) {
          throw new Error(`第 ${index + 1} 条 provider 的 interval 必须是正整数`);
        }
        interval = String(parsedInterval);
      }
      const hasData = name !== "" || url !== "" || path !== "" || interval !== "" || type !== "http";
      if (!hasData) {
        return null;
      }
      if (name === "" || url === "") {
        throw new Error(`第 ${index + 1} 条 provider 需要同时填写 name 和 url`);
      }
      return {
        name,
        type,
        url,
        path,
        interval
      };
    },
    normalizeProxyProviderEntries(entries, options = {}) {
      if (!Array.isArray(entries)) {
        throw new Error("proxy providers 必须是 JSON 数组");
      }
      const includeEmpty = options.includeEmpty === true;
      const normalized = [];
      entries.forEach((item, index) => {
        const normalizedItem = this.normalizeProxyProviderEntry(item, index);
        if (normalizedItem) {
          normalized.push(normalizedItem);
        }
      });
      if (normalized.length === 0 && includeEmpty) {
        return [this.createEmptyProxyProviderEntry()];
      }
      return normalized;
    },
    refreshProxyProvidersEditorFromEntries() {
      const providers = this.buildProxyProvidersArrayFromEntries();
      this.proxyProvidersJsonInput = JSON.stringify(providers, null, 2);
    },
    verifyProxyProvidersJsonWriteback() {
      const rawContent = typeof this.proxyProvidersJsonInput === "string" ? this.proxyProvidersJsonInput.trim() : "";
      if (rawContent === "") {
        this.clearProxyProviderEntries();
        this.notifySuccess("JSON 为空，已清空 Providers");
        return;
      }
      let parsed;
      try {
        parsed = JSON.parse(rawContent);
      } catch (error) {
        this.notifyError("JSON 解析失败，请检查语法");
        return;
      }
      try {
        this.form.proxyProviderEntries = this.normalizeProxyProviderEntries(parsed, {includeEmpty: true});
        this.syncProxyProvidersStringFromEntries();
        this.refreshProxyProvidersEditorFromEntries();
        this.notifySuccess("JSON 校验通过，已回填到引导输入区");
      } catch (error) {
        this.notifyError(error.message || "JSON 校验失败");
      }
    },
    parseProxyProvidersJson(content = this.form.proxyProviders) {
      if (typeof content !== "string") {
        return null;
      }
      content = content.trim();
      if (content === "") {
        return [];
      }
      try {
        return JSON.parse(content);
      } catch (e) {
        try {
          return JSON.parse(content.replace(/'/g, '"'));
        } catch (err) {
          return null;
        }
      }
    },
    syncProxyProviderEntriesFromProxyProvidersString() {
      const parsed = this.parseProxyProvidersJson();
      if (parsed === null || !Array.isArray(parsed)) {
        return false;
      }
      try {
        this.form.proxyProviderEntries = this.normalizeProxyProviderEntries(parsed, {includeEmpty: true});
        this.refreshProxyProvidersEditorFromEntries();
        return true;
      } catch (error) {
        return false;
      }
    },
    validateProxyProviderEntries() {
      try {
        this.normalizeProxyProviderEntries(this.form.proxyProviderEntries, {includeEmpty: true});
        return true;
      } catch (error) {
        this.notifyError(error.message || "Providers 参数不合法");
        return false;
      }
    },
    buildProxyProvidersArrayFromEntries() {
      let normalized = [];
      try {
        normalized = this.normalizeProxyProviderEntries(this.form.proxyProviderEntries);
      } catch (error) {
        normalized = [];
      }
      return normalized.map(item => {
        const provider = {
          name: item.name,
          type: item.type === "" ? "http" : item.type,
          url: item.url
        };
        if (item.path !== "") {
          provider.path = item.path;
        }
        if (item.interval !== "") {
          provider.interval = parseInt(item.interval, 10);
        }
        return provider;
      });
    },
    syncProxyProvidersStringFromEntries() {
      const providers = this.buildProxyProvidersArrayFromEntries();
      this.form.proxyProviders = providers.length === 0 ? "" : JSON.stringify(providers);
      return providers;
    },
    encodeProxyProvidersForRequest(proxyProviders) {
      return encodeURIComponent(proxyProviders);
    },
    decodeProxyProvidersFromRequest(proxyProviders) {
      return proxyProviders.replace(/%25([0-9A-Fa-f]{2})/g, "%$1");
    },
    getFirstParamValue(param, keys) {
      for (const key of keys) {
        const value = param.get(key);
        if (value !== null && value !== "") {
          return value;
        }
      }
      return "";
    },
    parseTruthyParam(value) {
      if (typeof value !== "string") {
        return false;
      }
      const normalized = value.trim().toLowerCase();
      return normalized === "true" || normalized === "1" || normalized === "yes";
    },
    parseBooleanParam(value) {
      if (typeof value !== "string") {
        return null;
      }
      const normalized = value.trim().toLowerCase();
      if (normalized === "true" || normalized === "1" || normalized === "yes" || normalized === "on") {
        return true;
      }
      if (normalized === "false" || normalized === "0" || normalized === "no" || normalized === "off") {
        return false;
      }
      return null;
    },
    parseBase36Mask(value) {
      if (typeof value !== "string" || value.trim() === "") {
        return 0;
      }
      let acc = 0;
      const text = value.trim().toLowerCase();
      for (let i = 0; i < text.length; i++) {
        const ch = text[i];
        let digit = -1;
        if (ch >= "0" && ch <= "9") {
          digit = ch.charCodeAt(0) - 48;
        } else if (ch >= "a" && ch <= "z") {
          digit = ch.charCodeAt(0) - 87;
        } else {
          return 0;
        }
        acc = acc * 36 + digit;
      }
      return acc;
    },
    isCompactDigestMode(rawMode) {
      if (typeof rawMode !== "string") {
        return false;
      }
      const mode = rawMode.trim().toLowerCase();
      return mode === "1" || mode === "true" || mode === "yes";
    },
    looksLikeQueryString(query) {
      if (typeof query !== "string" || query.trim() === "" || query.includes("\0")) {
        return false;
      }
      const content = query.startsWith("?") ? query.slice(1) : query;
      if (!content.includes("=")) {
        return false;
      }
      const firstToken = content.split("&")[0];
      const eqPos = firstToken.indexOf("=");
      if (eqPos <= 0) {
        return false;
      }
      return /^[A-Za-z0-9_.-]+$/.test(firstToken.slice(0, eqPos));
    },
    decodeBase64ToUtf8(content) {
      try {
        const normalized = content.replace(/-/g, "+").replace(/_/g, "/").replace(/ /g, "+").replace(/\r?\n/g, "");
        const padded = normalized + "===".slice((normalized.length + 3) % 4);
        const binary = window.atob(padded);
        let escaped = "";
        for (let i = 0; i < binary.length; i++) {
          escaped += "%" + binary.charCodeAt(i).toString(16).padStart(2, "0");
        }
        return decodeURIComponent(escaped);
      } catch (e) {
        return "";
      }
    },
    decodeBase64ToBytes(content) {
      try {
        const normalized = content.replace(/-/g, "+").replace(/_/g, "/").replace(/ /g, "+").replace(/\r?\n/g, "");
        const padded = normalized + "===".slice((normalized.length + 3) % 4);
        const binary = window.atob(padded);
        const bytes = new Uint8Array(binary.length);
        for (let i = 0; i < binary.length; i++) {
          bytes[i] = binary.charCodeAt(i);
        }
        return bytes;
      } catch (e) {
        return null;
      }
    },
    inflatePackedDigestQuery(base64Content) {
      const bytes = this.decodeBase64ToBytes(base64Content);
      if (!bytes) {
        return "";
      }
      const tryInflate = (fn) => {
        try {
          const output = fn(bytes, {to: "string"});
          return this.looksLikeQueryString(output) ? output : "";
        } catch (e) {
          return "";
        }
      };
      return tryInflate(pako.inflate) || tryInflate(pako.inflateRaw) || "";
    },
    encodeBytesToBase64Url(bytes) {
      try {
        let binary = "";
        const chunkSize = 0x8000;
        for (let i = 0; i < bytes.length; i += chunkSize) {
          binary += String.fromCharCode.apply(null, bytes.subarray(i, i + chunkSize));
        }
        return window.btoa(binary).replace(/\+/g, "-").replace(/\//g, "_").replace(/=+$/g, "");
      } catch (e) {
        return "";
      }
    },
    encodeUtf8ToBase64Url(content) {
      try {
        const utf8Binary = encodeURIComponent(content).replace(/%([0-9A-F]{2})/g, (_, p1) => String.fromCharCode(parseInt(p1, 16)));
        return window.btoa(utf8Binary).replace(/\+/g, "-").replace(/\//g, "_").replace(/=+$/g, "");
      } catch (e) {
        return content;
      }
    },
    encodeUtf8ToDeflateRawBase64Url(content) {
      try {
        const compressed = pako.deflateRaw(content, {level: 9});
        return this.encodeBytesToBase64Url(compressed);
      } catch (e) {
        return "";
      }
    },
    decodePackedDigestQuery(rawPackedQuery) {
      if (typeof rawPackedQuery !== "string" || rawPackedQuery.trim() === "") {
        return "";
      }
      let packed = rawPackedQuery.trim();
      try {
        packed = decodeURIComponent(packed);
      } catch (e) {
        // keep original string
      }
      if (packed.startsWith("?")) {
        packed = packed.slice(1);
      }
      if (this.looksLikeQueryString(packed)) {
        return packed;
      }
      const decodedBase64 = this.decodeBase64ToUtf8(packed);
      if (decodedBase64 !== "" && this.looksLikeQueryString(decodedBase64)) {
        return decodedBase64;
      }
      const inflatedQuery = this.inflatePackedDigestQuery(packed);
      if (inflatedQuery !== "") {
        return inflatedQuery;
      }
      return "";
    },
    buildCompactDigestQueryFromSubQuery(subQuery) {
      const fullParam = new URLSearchParams(subQuery);
      const compactParam = new URLSearchParams();
      compactParam.set("m", "1");

      let trueMask = 0;
      let falseMask = 0;
      const boolIndexMap = digestCompactBoolSpecs.reduce((acc, item, index) => {
        acc[item.key] = index;
        return acc;
      }, {});

      for (const [key, value] of fullParam.entries()) {
        if (key === "filename") {
          continue;
        }

        if (Object.prototype.hasOwnProperty.call(boolIndexMap, key)) {
          const parsedBool = this.parseBooleanParam(value);
          if (parsedBool === null) {
            continue;
          }
          const idx = boolIndexMap[key];
          const defaultValue = digestCompactBoolSpecs[idx].defaultValue;
          if (parsedBool === defaultValue) {
            continue;
          }
          if (parsedBool) {
            trueMask |= (1 << idx);
          } else {
            falseMask |= (1 << idx);
          }
          continue;
        }

        if (key === "config" && value === defaultRemoteConfig) {
          continue;
        }

        const aliasKey = digestCompactAliasByLongKey[key] || key;
        compactParam.set(aliasKey, value);
      }

      if (trueMask !== 0) {
        compactParam.set("bt", trueMask.toString(36));
      }
      if (falseMask !== 0) {
        compactParam.set("bf", falseMask.toString(36));
      }
      return compactParam.toString();
    },
    expandCompactDigestParams(param, options = {}) {
      const applyDefaults = options.applyDefaults !== false;
      const expanded = new URLSearchParams();
      const compactMode = this.isCompactDigestMode(param.get("m"));
      if (!compactMode) {
        for (const [key, value] of param.entries()) {
          expanded.set(key, value);
        }
        return expanded;
      }

      for (const [key, value] of param.entries()) {
        if (key === "m" || key === "bt" || key === "bf") {
          continue;
        }
        const longKey = digestCompactAliasByShortKey[key] || key;
        expanded.set(longKey, value);
      }

      const trueMask = this.parseBase36Mask(param.get("bt") || "");
      const falseMask = this.parseBase36Mask(param.get("bf") || "");
      for (let i = 0; i < digestCompactBoolSpecs.length; i++) {
        const spec = digestCompactBoolSpecs[i];
        if (expanded.has(spec.key)) {
          continue;
        }
        const bit = 1 << i;
        if ((trueMask & bit) !== 0) {
          expanded.set(spec.key, "true");
          continue;
        }
        if ((falseMask & bit) !== 0) {
          expanded.set(spec.key, "false");
          continue;
        }
        if (applyDefaults) {
          expanded.set(spec.key, spec.defaultValue ? "true" : "false");
        }
      }

      if (applyDefaults && !expanded.has("config")) {
        expanded.set("config", defaultRemoteConfig);
      }
      return expanded;
    },
    mergeDigestQueryParams(rawParam) {
      const mergedParam = new URLSearchParams();
      const packedQuery = rawParam.get("q");
      if (packedQuery) {
        const unpackedQuery = this.decodePackedDigestQuery(packedQuery);
        if (unpackedQuery !== "") {
          const digestParam = new URLSearchParams(unpackedQuery.startsWith("?") ? unpackedQuery.slice(1) : unpackedQuery);
          const expandedDigestParam = this.expandCompactDigestParams(digestParam, {applyDefaults: true});
          for (const [key, value] of expandedDigestParam.entries()) {
            if (key !== "q") {
              mergedParam.set(key, value);
            }
          }
        }
      }
      const expandedRawParam = this.expandCompactDigestParams(rawParam, {applyDefaults: false});
      for (const [key, value] of expandedRawParam.entries()) {
        if (key !== "q") {
          mergedParam.set(key, value);
        }
      }
      return mergedParam;
    },
    buildSubQueryParts(sourceSub, options = {}) {
      const includeFilename = options.includeFilename !== false;
      let queryParts = [];
      if (this.form.clientType.includes("&")) {
        const clientTypeParts = this.form.clientType.split("&");
        queryParts.push("target=" + clientTypeParts[0]);
        clientTypeParts.slice(1).forEach(part => {
          if (part !== "") {
            queryParts.push(part);
          }
        });
      } else {
        queryParts.push("target=" + this.form.clientType);
      }
      queryParts.push("url=" + encodeURIComponent(sourceSub));
      queryParts.push("insert=" + this.form.insert);
      if (this.form.remoteConfig !== "") {
        queryParts.push("config=" + encodeURIComponent(this.form.remoteConfig));
      }
      if (this.form.excludeRemarks !== "") {
        queryParts.push("exclude=" + encodeURIComponent(this.form.excludeRemarks));
      }
      if (this.form.includeRemarks !== "") {
        queryParts.push("include=" + encodeURIComponent(this.form.includeRemarks));
      }
      if (includeFilename && this.form.filename !== "") {
        queryParts.push("filename=" + encodeURIComponent(this.form.filename));
      }
      if (this.form.rename !== "") {
        queryParts.push("rename=" + encodeURIComponent(this.form.rename));
      }
      if (this.form.interval !== "") {
        queryParts.push("interval=" + encodeURIComponent(this.form.interval * 86400));
      }
      if (this.form.devid !== "") {
        queryParts.push("dev_id=" + encodeURIComponent(this.form.devid));
      }
      if (this.form.appendType) {
        queryParts.push("append_type=" + this.form.appendType.toString());
      }
      if (this.form.tls13) {
        queryParts.push("tls13=" + this.form.tls13.toString());
      }
      if (this.form.sort) {
        queryParts.push("sort=" + this.form.sort.toString());
      }
      if (this.form.useDialer) {
        queryParts.push("use_dialer=true");
        if (this.form.dialerGroupName.trim() !== "") {
          queryParts.push("dialer_group_name=" + encodeURIComponent(this.form.dialerGroupName.trim()));
        }
        if (this.form.applyDialerTo.trim() !== "") {
          queryParts.push("apply_dialer_to=" + encodeURIComponent(this.form.applyDialerTo.trim()));
        }
      }
      if (this.form.proxyProviders.trim() !== "") {
        queryParts.push("proxy_providers=" + this.encodeProxyProvidersForRequest(this.form.proxyProviders.trim()));
      }
      queryParts.push("emoji=" + this.form.emoji.toString());
      queryParts.push("list=" + this.form.nodeList.toString());
      queryParts.push("xudp=" + this.form.xudp.toString());
      queryParts.push("udp=" + this.form.udp.toString());
      queryParts.push("tfo=" + this.form.tfo.toString());
      queryParts.push("expand=" + this.form.expand.toString());
      queryParts.push("scv=" + this.form.scv.toString());
      queryParts.push("fdn=" + this.form.fdn.toString());
      if (this.form.clientType.includes("surge") && this.form.tpl.surge.doh === true) {
        queryParts.push("surge.doh=true");
      }
      if (this.form.clientType === "clash") {
        if (this.form.tpl.clash.doh === true) {
          queryParts.push("clash.doh=true");
        }
        queryParts.push("new_name=" + this.form.new_name.toString());
      }
      if (this.form.clientType === "singbox" && this.form.tpl.singbox.ipv6 === true) {
        queryParts.push("singbox.ipv6=1");
      }
      return queryParts;
    },
    buildDigestUrl(backend, subQuery) {
      let digestParams = [];
      if (this.form.filename.trim() !== "") {
        digestParams.push("a=" + encodeURIComponent(this.form.filename.trim()));
      }

      const compactQuery = this.buildCompactDigestQueryFromSubQuery(subQuery);
      const candidateQueryList = compactQuery !== "" ? [compactQuery, subQuery] : [subQuery];
      let packedQuery = "";
      for (const queryText of candidateQueryList) {
        if (!queryText) {
          continue;
        }
        const compressedPackedQuery = this.encodeUtf8ToDeflateRawBase64Url(queryText);
        if (compressedPackedQuery !== "" && (packedQuery === "" || compressedPackedQuery.length < packedQuery.length)) {
          packedQuery = compressedPackedQuery;
        }
        const plainPackedQuery = this.encodeUtf8ToBase64Url(queryText);
        if (plainPackedQuery !== "" && (packedQuery === "" || plainPackedQuery.length < packedQuery.length)) {
          packedQuery = plainPackedQuery;
        }
      }
      if (packedQuery === "" && compactQuery !== "") {
        packedQuery = compactQuery;
      } else if (packedQuery === "") {
        packedQuery = subQuery;
      }
      digestParams.push("q=" + encodeURIComponent(packedQuery));
      return backend + "/digest?" + digestParams.join("&");
    },
    makeUrl() {
      if (this.form.sourceSubUrl === "" || this.form.clientType === "") {
        this.notifyError("订阅链接与客户端为必填项");
        return false;
      }
      if (!this.ensureDialerBackendSupported(true)) {
        return false;
      }
      if (!this.validateProxyProviderEntries()) {
        return false;
      }
      this.syncProxyProvidersStringFromEntries();
      let backend =
          this.form.customBackend === ""
              ? defaultBackend
              : this.form.customBackend;
      let sourceSub = this.form.sourceSubUrl;
      sourceSub = sourceSub.replace(/(\n|\r|\n\r)/g, "|");
      const subQuery = this.buildSubQueryParts(sourceSub, {includeFilename: !this.form.useDigest}).join("&");
      this.customSubUrl = this.form.useDigest ? this.buildDigestUrl(backend, subQuery) : backend + "/sub?" + subQuery;
      this.$copyText(this.customSubUrl);
      this.notifySuccess("定制订阅已复制到剪贴板");
    },
    confirmUploadConfig() {
      this.loading2 = true;
      let data = new FormData();
      data.append("config", encodeURIComponent(this.uploadConfig));
      this.$axios
          .post(configUploadBackend, data, {
            header: {
              "Content-Type": "application/form-data; charset=utf-8"
            }
          })
          .then(res => {
            if (res.data.code === 0 && res.data.data !== "") {
              this.notifySuccess(
                  "远程配置上传成功，配置链接已复制到剪贴板"
              );
              this.form.remoteConfig = res.data.data;
              this.$copyText(this.form.remoteConfig);
              this.dialogUploadConfigVisible = false;
            } else {
              this.notifyError("远程配置上传失败: " + res.data.msg);
            }
          })
          .catch(() => {
            this.notifyError("远程配置上传失败");
          })
          .finally(() => {
            this.loading2 = false;
          });
    },
    analyzeUrl() {
      if (this.loadConfig.indexOf("target=") !== -1 || this.loadConfig.indexOf("q=") !== -1) {
        return this.loadConfig;
      } else {
        this.loading3 = true;
        return (async () => {
          try {
            let response = await fetch(this.loadConfig, {
              method: "GET",
              redirect: "follow",
            });
            return response.url;
          } catch (e) {
            this.notifyError("解析短链接失败，请检查短链接服务端是否配置跨域：" + e)
          } finally {
            this.loading3 = false;
          }
        })();
      }
    },
    confirmLoadConfig() {
      if (this.loadConfig.trim() === "" || !this.loadConfig.trim().includes("http")) {
        this.notifyError("待解析的订阅链接不合法");
        return false;
      }
      (async () => {
        let url
        try {
          url = new URL(await this.analyzeUrl())
        } catch (error) {
          this.notifyError("请输入正确的订阅地址!");
          return;
        }
        this.form.customBackend = url.origin
        const rawParam = new URLSearchParams(url.search);
        const aliasParam = rawParam.get("a");
        const isDigestPath = url.pathname === "/digest";
        if (isDigestPath) {
          this.form.useDigest = true;
        } else if (url.pathname === "/sub") {
          this.form.useDigest = false;
        }
        let param = this.mergeDigestQueryParams(rawParam);
        if (param.get("target")) {
          let target = param.get("target");
          if (target === 'surge' && param.get("ver")) {
            // 类型为surge,有ver
            this.form.clientType = target + "&ver=" + param.get("ver");
          } else if (target === 'surge') {
            //类型为surge,没有ver
            this.form.clientType = target + "&ver=4"
          } else {
            //类型为其他
            this.form.clientType = target;
          }
        }
        if (param.get("url")) {
          this.form.sourceSubUrl = param.get("url");
        }
        if (param.get("insert")) {
          this.form.insert = param.get("insert") === 'true';
        }
        if (param.get("config")) {
          this.form.remoteConfig = param.get("config");
        }
        if (param.get("exclude")) {
          this.form.excludeRemarks = param.get("exclude");
        }
        if (param.get("include")) {
          this.form.includeRemarks = param.get("include");
        }
        if (isDigestPath && aliasParam) {
          this.form.filename = aliasParam;
        } else if (param.get("filename")) {
          this.form.filename = param.get("filename");
        } else if (aliasParam) {
          this.form.filename = aliasParam;
        }
        if (param.get("rename")) {
          this.form.rename = param.get("rename");
        }
        if (param.get("interval")) {
          this.form.interval = Math.ceil(param.get("interval") / 86400);
        }
        if (param.get("dev_id")) {
          this.form.devid = param.get("dev_id");
        }
        if (param.get("append_type")) {
          this.form.appendType = param.get("append_type") === 'true';
        }
        if (param.get("tls13")) {
          this.form.tls13 = param.get("tls13");
        }
        if (param.get("xudp")) {
          this.form.xudp = param.get("xudp") === 'true';
        }
        if (param.get("sort")) {
          this.form.sort = param.get("sort") === 'true';
        }
        const useDialerParam = this.getFirstParamValue(param, ["use_dialer", "useDialer"]);
        if (useDialerParam !== "") {
          this.form.useDialer = this.parseTruthyParam(useDialerParam);
        }
        const dialerGroupNameParam = this.getFirstParamValue(param, ["dialer_group_name", "dialerGroupName"]);
        if (dialerGroupNameParam !== "") {
          this.form.dialerGroupName = dialerGroupNameParam;
        }
        const applyDialerToParam = this.getFirstParamValue(param, ["apply_dialer_to", "applyDialerTo"]);
        if (applyDialerToParam !== "") {
          this.form.applyDialerTo = applyDialerToParam;
        }
        const proxyProvidersParam = this.getFirstParamValue(param, ["proxy_providers", "proxy-providers", "proxyProviders"]);
        if (proxyProvidersParam !== "") {
          this.form.proxyProviders = this.decodeProxyProvidersFromRequest(proxyProvidersParam);
          if (!this.syncProxyProviderEntriesFromProxyProvidersString()) {
            this.notifyError("解析 proxy_providers 失败，请检查参数格式");
            return;
          }
        }
        if (param.get("emoji")) {
          this.form.emoji = param.get("emoji") === 'true';
        }
        if (param.get("list")) {
          this.form.nodeList = param.get("list") === 'true';
        }
        if (param.get("udp")) {
          this.form.udp = param.get("udp") === 'true';
        }
        if (param.get("tfo")) {
          this.form.tfo = param.get("tfo") === 'true';
        }
        if (param.get("expand")) {
          this.form.expand = param.get("expand") === 'true';
        }
        if (param.get("scv")) {
          this.form.scv = param.get("scv") === 'true';
        }
        if (param.get("fdn")) {
          this.form.fdn = param.get("fdn") === 'true';
        }
        if (param.get("surge.doh")) {
          this.form.tpl.surge.doh = param.get("surge.doh") === 'true';
        }
        if (param.get("clash.doh")) {
          this.form.tpl.clash.doh = param.get("clash.doh") === 'true';
        }
        if (param.get("new_name")) {
          this.form.new_name = param.get("new_name") === 'true';
        }
        if (param.get("singbox.ipv6")) {
          this.form.tpl.singbox.ipv6 = param.get("singbox.ipv6") === '1';
        }
        this.dialogLoadConfigVisible = false;
        this.notifySuccess("长/短链接已成功解析为订阅信息");
      })();
    },
    renderPost() {
      if (!this.ensureDialerBackendSupported(true)) {
        return null;
      }
      if (!this.validateProxyProviderEntries()) {
        return null;
      }
      this.syncProxyProvidersStringFromEntries();
      let data = new FormData();
      data.append("target", encodeURIComponent(this.form.clientType));
      data.append("url", encodeURIComponent(this.form.sourceSubUrl));
      data.append("config", encodeURIComponent(this.form.remoteConfig));
      data.append("exclude", encodeURIComponent(this.form.excludeRemarks));
      data.append("include", encodeURIComponent(this.form.includeRemarks));
      data.append("rename", encodeURIComponent(this.form.rename));
      data.append("tls13", encodeURIComponent(this.form.tls13.toString()));
      data.append("xudp", encodeURIComponent(this.form.xudp.toString()));
      data.append("emoji", encodeURIComponent(this.form.emoji.toString()));
      data.append("list", encodeURIComponent(this.form.nodeList.toString()));
      data.append("udp", encodeURIComponent(this.form.udp.toString()));
      data.append("tfo", encodeURIComponent(this.form.tfo.toString()));
      data.append("expand", encodeURIComponent(this.form.expand.toString()));
      data.append("scv", encodeURIComponent(this.form.scv.toString()));
      data.append("fdn", encodeURIComponent(this.form.fdn.toString()));
      data.append("sdoh", encodeURIComponent(this.form.tpl.surge.doh.toString()));
      data.append("cdoh", encodeURIComponent(this.form.tpl.clash.doh.toString()));
      data.append("newname", encodeURIComponent(this.form.new_name.toString()));
      data.append("use_dialer", encodeURIComponent(this.form.useDialer.toString()));
      data.append("dialer_group_name", encodeURIComponent(this.form.dialerGroupName));
      data.append("apply_dialer_to", encodeURIComponent(this.form.applyDialerTo));
      data.append("proxy_providers", this.encodeProxyProvidersForRequest(this.form.proxyProviders));
      return data;
    },
    confirmUploadScript() {
      if (this.form.sourceSubUrl.trim() === "") {
        this.notifyError("订阅链接不能为空");
        return false;
      }
      this.loading2 = true;
      let data = this.renderPost();
      if (!data) {
        this.loading2 = false;
        return false;
      }
      data.append("sortscript", encodeURIComponent(this.uploadScript));
      data.append("filterscript", encodeURIComponent(this.uploadFilter));
      this.$axios
          .post(configScriptBackend, data, {
            header: {
              "Content-Type": "application/form-data; charset=utf-8"
            }
          })
          .then(res => {
            if (res.data.code === 0 && res.data.data !== "") {
              this.notifySuccess(
                  "自定义JS上传成功，订阅链接已复制到剪贴板（IOS设备和Safari浏览器不支持自动复制API，需手动点击复制按钮）"
              );
              this.customSubUrl = res.data.data;
              this.$copyText(res.data.data);
              this.dialogUploadConfigVisible = false;
              this.btnBoolean = true;
            } else {
              this.notifyError("自定义JS上传失败: " + res.data.msg);
            }
          })
          .catch(() => {
            this.notifyError("自定义JS上传失败");
          })
          .finally(() => {
            this.loading2 = false;
          })
    },
    getBackendVersion() {
      this.$axios
          .get(
              this.form.customBackend + "/version"
          )
          .then(res => {
            this.backendVersion = res.data.replace(/backend\n$/gm, "");
            this.backendVersion = this.backendVersion.replace("subconverter", "SubConverter");
          })
          .catch(() => {
            this.backendVersion = "不可用";
          });
    }
  }
};
</script>

<style lang="scss">
.subconverter-page {
  --paper: #f3f2ed;
  --ink: #1b2430;
  --muted-ink: #5d6776;
  --panel: #fcfbf8;
  --panel-border: #dad6cc;
  --accent: #2f5a73;
  --accent-soft: #dfe9ef;
  --accent-strong: #1f4257;
  --shadow: 0 14px 34px rgba(28, 33, 44, 0.09);
  min-height: 100vh;
  padding: 24px 14px 40px;
  color: var(--ink);
  background:
      radial-gradient(circle at 12% 10%, rgba(47, 90, 115, 0.16), transparent 40%),
      radial-gradient(circle at 94% 2%, rgba(47, 90, 115, 0.1), transparent 34%),
      linear-gradient(158deg, #ebeae4 0%, #f5f2ea 45%, #f0eee7 100%);
  font-family: "IBM Plex Sans", "Noto Sans SC", "PingFang SC", "Microsoft YaHei", sans-serif;
}

.subconverter-page .page-row {
  max-width: 1180px;
  margin: 0 auto;
}

.subconverter-page .form-shell,
.subconverter-page .primary-form {
  width: 100%;
}

.subconverter-page .converter-card {
  border: 1px solid var(--panel-border);
  border-radius: 20px;
  overflow: hidden;
  background: var(--panel);
  box-shadow: var(--shadow);
  animation: cardRise 380ms ease-out;
}

.subconverter-page .converter-card > .el-card__header {
  background: linear-gradient(148deg, #f9f8f4 0%, #ede9df 100%);
  border-bottom: 1px solid var(--panel-border);
  padding: 16px 22px;
}

.subconverter-page .converter-card > .el-card__body {
  padding: 18px 22px 24px;
}

.subconverter-page .converter-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.subconverter-page .header-side {
  display: flex;
  align-items: center;
  gap: 10px;
  min-width: 84px;
}

.subconverter-page .header-side-right {
  justify-content: flex-end;
}

.subconverter-page .header-title {
  flex: 1;
  min-width: 0;
  text-align: center;
}

.subconverter-page .header-title-cn {
  letter-spacing: 0.03em;
  font-size: 28px;
  font-weight: 700;
  color: #213446;
  font-family: "Source Serif 4", "Noto Serif SC", "Songti SC", serif;
}

.subconverter-page .header-title-en {
  margin-top: 4px;
  font-size: 11px;
  letter-spacing: 0.14em;
  color: var(--muted-ink);
  text-transform: uppercase;
}

.subconverter-page .header-icon {
  font-size: 22px;
  color: var(--accent-strong);
  transition: transform 0.2s ease, color 0.2s ease;
}

.subconverter-page .header-icon:hover {
  transform: translateY(-1px);
  color: var(--accent);
}

.subconverter-page .info-band {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 12px;
  margin: 0 0 18px;
}

.subconverter-page .info-band-item {
  background: #f3f1eb;
  border: 1px solid #dfdbd0;
  border-radius: 12px;
  padding: 12px 14px;
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.subconverter-page .info-label {
  font-size: 11px;
  color: var(--muted-ink);
  letter-spacing: 0.04em;
}

.subconverter-page .info-band-item strong {
  color: #1f3f53;
  font-size: 13px;
}

.subconverter-page .primary-form .el-form-item {
  margin-bottom: 14px;
}

.subconverter-page .primary-form .el-form-item__label {
  color: #2c3b4a;
  font-weight: 600;
}

.subconverter-page .section-divider {
  margin: 10px 0 16px;
}

.subconverter-page .section-divider .el-divider__text {
  background: var(--panel);
  color: #2c4658;
  font-weight: 700;
  letter-spacing: 0.08em;
}

.subconverter-page .dense-alert {
  margin: 2px 0 12px;
}

.subconverter-page .remote-preset-item {
  margin-top: -4px;
}

.subconverter-page .advanced-collapse {
  border: 1px solid #d7dde4;
  border-radius: 12px;
  overflow: hidden;
  background: #f7fafc;
}

.subconverter-page .advanced-collapse .el-collapse-item__header {
  height: auto;
  line-height: normal;
  border-bottom: none;
  background: transparent;
  padding: 10px 12px;
}

.subconverter-page .advanced-collapse .el-collapse-item__wrap {
  border-bottom: none;
  background: transparent;
}

.subconverter-page .advanced-toggle-header {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.subconverter-page .advanced-toggle-label {
  font-weight: 700;
  font-size: 14px;
  color: #2f4758;
  letter-spacing: 0.03em;
}

.subconverter-page .advanced-toggle-action {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 13px;
  color: #5f7384;
}

.subconverter-page .copy-content .el-input__inner {
  font-family: "JetBrains Mono", "SFMono-Regular", Menlo, Monaco, Consolas, monospace;
  font-size: 12px;
}

.subconverter-page .provider-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 12px;
}

.subconverter-page .provider-column {
  border: 1px solid #dfdbd1;
  border-radius: 12px;
  background: #f9f8f4;
  padding: 10px;
}

.subconverter-page .provider-json-column {
  display: flex;
  flex-direction: column;
}

.subconverter-page .provider-guide-tip,
.subconverter-page .provider-json-tip {
  font-size: 12px;
  color: #647080;
  line-height: 1.5;
}

.subconverter-page .provider-guide-tip {
  margin-bottom: 8px;
}

.subconverter-page .provider-card {
  border: 1px solid #e5e1d8;
  border-radius: 10px;
  background: #fffefb;
  padding: 10px;
  margin-bottom: 10px;
}

.subconverter-page .provider-actions {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  margin-top: 8px;
}

.subconverter-page .provider-json-editor {
  position: relative;
  flex: 1;
  min-height: 220px;
}

.subconverter-page .provider-json-overlay {
  position: absolute;
  inset: 0;
  margin: 0;
  overflow: auto;
  padding: 10px 12px;
  border: 1px solid #d8e0e8;
  border-radius: 10px;
  background: #f5f8fb;
  color: #2f4050;
  font-family: "JetBrains Mono", "SFMono-Regular", Menlo, Monaco, Consolas, monospace;
  font-size: 12px;
  line-height: 1.5;
  white-space: pre-wrap;
  word-break: break-word;
  pointer-events: none;
}

.subconverter-page .provider-json-textarea {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  padding: 10px 12px;
  border: 1px solid #d8e0e8;
  border-radius: 10px;
  resize: none;
  background: transparent;
  color: transparent;
  caret-color: #2f4f68;
  font-family: "JetBrains Mono", "SFMono-Regular", Menlo, Monaco, Consolas, monospace;
  font-size: 12px;
  line-height: 1.5;
  outline: none;
}

.subconverter-page .provider-json-textarea::placeholder {
  color: #7e92a6;
}

.subconverter-page .provider-json-textarea::selection {
  background: rgba(126, 174, 208, 0.35);
  color: transparent;
}

.subconverter-page .provider-json-editor.is-error .provider-json-overlay,
.subconverter-page .provider-json-editor.is-error .provider-json-textarea {
  border-color: #b85f6c;
}

.subconverter-page .provider-json-title {
  font-size: 12px;
  color: #2d4a5e;
  margin-bottom: 8px;
  letter-spacing: 0.04em;
  font-weight: 700;
}

.subconverter-page .provider-json-tip {
  margin-top: 8px;
}

.subconverter-page .provider-expression-hint {
  margin: 0 0 8px;
  font-size: 12px;
  line-height: 1.5;
  color: #5f7486;
}

.subconverter-page .provider-expression-error {
  color: #c4515f;
}

.subconverter-page .provider-expression-code {
  background: #e9eef3;
  border: 1px solid #d3dee7;
  border-radius: 6px;
  padding: 1px 6px;
  color: #2f4f67;
  font-family: "JetBrains Mono", "SFMono-Regular", Menlo, Monaco, Consolas, monospace;
  word-break: break-all;
}

.subconverter-page .provider-name-match {
  background: #ffdf95;
  color: #4a3610;
  border-radius: 3px;
  padding: 0 1px;
}

.subconverter-page .el-input__inner,
.subconverter-page .el-textarea__inner,
.subconverter-page .el-select .el-input__inner {
  border-radius: 10px;
}

.subconverter-page .el-input__inner:focus,
.subconverter-page .el-textarea__inner:focus {
  border-color: #7294ab;
}

.subconverter-page .el-button {
  border-radius: 10px;
}

.subconverter-page .el-dialog {
  border-radius: 14px;
}

.subconverter-page .el-dialog__header {
  border-bottom: 1px solid #e2e8ee;
  padding-bottom: 12px;
}

body.dark-mode .subconverter-page {
  --ink: #d7e4f0;
  --muted-ink: #9cb2c7;
  --panel: #111a23;
  --panel-border: #2e3b49;
  --accent: #74a8cc;
  --accent-strong: #91c2e3;
  --shadow: 0 16px 36px rgba(5, 8, 12, 0.45);
  background:
      radial-gradient(circle at 14% 8%, rgba(88, 140, 176, 0.24), transparent 42%),
      radial-gradient(circle at 92% 0%, rgba(76, 120, 152, 0.17), transparent 32%),
      linear-gradient(160deg, #080d12 0%, #0f1620 52%, #0a1118 100%);
}

body.dark-mode .subconverter-page .converter-card {
  background: #101922 !important;
  border-color: #314252 !important;
  box-shadow: var(--shadow);
}

body.dark-mode .subconverter-page .converter-card > .el-card__header {
  background: linear-gradient(150deg, #15212b 0%, #101820 100%) !important;
  border-bottom-color: #2d3d4c !important;
}

body.dark-mode .subconverter-page .converter-card > .el-card__body {
  background: #101922 !important;
}

body.dark-mode .subconverter-page .header-title-cn {
  color: #e4edf4;
}

body.dark-mode .subconverter-page .header-icon {
  color: #9ac2df;
}

body.dark-mode .subconverter-page .header-icon:hover {
  color: #bfd9ee;
}

body.dark-mode .subconverter-page .info-band-item {
  background: #16222d;
  border-color: #334658;
}

body.dark-mode .subconverter-page .info-band-item strong {
  color: #d3e4f2;
}

body.dark-mode .subconverter-page .primary-form .el-form-item__label,
body.dark-mode .subconverter-page .info-label,
body.dark-mode .subconverter-page .provider-guide-tip,
body.dark-mode .subconverter-page .provider-json-tip,
body.dark-mode .subconverter-page .provider-json-title {
  color: #9fb5c8 !important;
}

body.dark-mode .subconverter-page .section-divider .el-divider__text {
  background: #101922 !important;
  color: #bfd7ea !important;
}

body.dark-mode .subconverter-page .advanced-collapse {
  background: #14202a;
  border-color: #324556;
}

body.dark-mode .subconverter-page .advanced-collapse .el-collapse-item__header {
  background: #172530;
  border-bottom-color: transparent;
}

body.dark-mode .subconverter-page .advanced-collapse .el-collapse-item__wrap {
  background: #14202a;
}

body.dark-mode .subconverter-page .advanced-toggle-label {
  color: #d2e2ef;
}

body.dark-mode .subconverter-page .advanced-toggle-action {
  color: #9db4c8;
}

body.dark-mode .subconverter-page .provider-column {
  background: #121d27;
  border-color: #314252;
}

body.dark-mode .subconverter-page .provider-card {
  background: #17232e;
  border-color: #33485c;
}

body.dark-mode .subconverter-page .provider-expression-hint {
  color: #9db4c8;
}

body.dark-mode .subconverter-page .provider-expression-error {
  color: #f08a97;
}

body.dark-mode .subconverter-page .provider-expression-code {
  background: #1e2f3d;
  border-color: #3d5569;
  color: #d3e4f1;
}

body.dark-mode .subconverter-page .provider-name-match {
  background: #ecc97a;
  color: #2a1f08;
}

body.dark-mode .subconverter-page .provider-json-overlay {
  background: #172631;
  border-color: #3d566a;
  color: #d8e6f2;
}

body.dark-mode .subconverter-page .provider-json-textarea {
  border-color: #3d566a;
  caret-color: #e8f2fa;
}

body.dark-mode .subconverter-page .provider-json-textarea::placeholder {
  color: #8ca5ba;
}

body.dark-mode .subconverter-page .el-alert {
  background-color: #1a2733 !important;
  border-color: #31495d !important;
}

body.dark-mode .subconverter-page .el-alert__title,
body.dark-mode .subconverter-page .el-alert__description,
body.dark-mode .subconverter-page .el-alert__icon {
  color: #c1d6e7 !important;
}

body.dark-mode .subconverter-page .el-input__inner,
body.dark-mode .subconverter-page .el-textarea__inner,
body.dark-mode .subconverter-page .el-select .el-input__inner {
  background: #1a2632 !important;
  border-color: #3d5367 !important;
  color: #e0ebf5 !important;
}

body.dark-mode .subconverter-page .el-input__inner:focus,
body.dark-mode .subconverter-page .el-textarea__inner:focus,
body.dark-mode .subconverter-page .el-select .el-input.is-focus .el-input__inner {
  border-color: #7eaed0 !important;
}

body.dark-mode .subconverter-page .el-input__inner::placeholder,
body.dark-mode .subconverter-page .el-textarea__inner::placeholder {
  color: rgba(196, 216, 232, 0.62) !important;
}

body.dark-mode .subconverter-page .el-button--default {
  background: #1a2732 !important;
  border-color: #3a5165 !important;
  color: #d4e5f3 !important;
}

body.dark-mode .subconverter-page .el-button--default:hover,
body.dark-mode .subconverter-page .el-button--default:focus {
  background: #223546 !important;
  border-color: #587993 !important;
  color: #e7f1f8 !important;
}

body.dark-mode .subconverter-page .el-button--primary {
  background: #2f5d7b !important;
  border-color: #4a7fa2 !important;
}

body.dark-mode .subconverter-page .el-button--success {
  background: #2f6b58 !important;
  border-color: #4b8b75 !important;
}

body.dark-mode .subconverter-page .el-button--danger {
  background: #7f3340 !important;
  border-color: #a74d5c !important;
}

.el-message.message-bottom-right {
  left: auto !important;
  right: 16px !important;
  top: auto !important;
  bottom: 16px !important;
  transform: none !important;
  margin: 0 !important;
}

@keyframes cardRise {
  from {
    opacity: 0;
    transform: translateY(8px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (max-width: 900px) {
  .subconverter-page {
    padding: 10px 8px 24px;
  }

  .subconverter-page .converter-card > .el-card__header,
  .subconverter-page .converter-card > .el-card__body {
    padding-left: 12px;
    padding-right: 12px;
  }

  .subconverter-page .header-title-cn {
    font-size: 22px;
  }

  .subconverter-page .header-title-en {
    font-size: 11px;
  }

  .subconverter-page .info-band {
    grid-template-columns: 1fr;
  }

  .subconverter-page .advanced-toggle-header {
    flex-wrap: wrap;
    gap: 6px;
  }

  .subconverter-page .header-side {
    min-width: auto;
    gap: 8px;
  }

  .subconverter-page .provider-grid {
    grid-template-columns: 1fr;
  }
}
</style>

