<template>
  <q-dialog ref="dialog" @hide="onHide">
    <q-card style="width: 90vw; max-width: 90vw">
      <q-bar>
        {{ title }}
        <q-space />
        <q-btn dense flat icon="close" v-close-popup>
          <q-tooltip class="bg-white text-primary">Close</q-tooltip>
        </q-btn>
      </q-bar>
      <q-stepper
        v-model="step"
        ref="stepper"
        alternative-labels
        header-nav
        color="primary"
        animated
      >
        <q-step
          :name="1"
          :error="!template.name && step > 1"
          title="General Settings"
          icon="settings"
        >
          <q-card flat>
            <q-card-section>
              <q-input
                label="Name"
                class="q-mb-none"
                outlined
                dense
                v-model="template.name"
                :rules="[(val) => !!val || '*Required']"
              />
            </q-card-section>

            <q-card-section>
              <q-toggle
                v-model="template.is_active"
                color="green"
                label="Enabled"
                left-label
              />
            </q-card-section>

            <div class="q-pl-md text-subtitle1">
              Email Settings (Overrides global email settings)
            </div>

            <q-card-section>
              <q-input
                label="Email From address"
                class="q-mb-sm"
                outlined
                dense
                v-model="template.email_from"
              />
            </q-card-section>

            <q-card-section class="row">
              <div class="col-2 q-mb-sm">Email recipients</div>
              <div class="col-4 q-mb-sm">
                <q-list dense v-if="template.email_recipients.length !== 0">
                  <q-item
                    v-for="email in template.email_recipients"
                    :key="email"
                    dense
                  >
                    <q-item-section>
                      <q-item-label>{{ email }}</q-item-label>
                    </q-item-section>
                    <q-item-section side>
                      <q-icon
                        class="cursor-pointer"
                        name="delete"
                        color="red"
                        @click="removeEmail(email)"
                      />
                    </q-item-section>
                  </q-item>
                </q-list>
                <q-list v-else>
                  <q-item-section>
                    <q-item-label>No recipients</q-item-label>
                  </q-item-section>
                </q-list>
              </div>
              <div class="col-3 q-mb-sm"></div>
              <div class="col-3 q-mb-sm">
                <q-btn
                  size="sm"
                  icon="fas fa-plus"
                  color="secondary"
                  label="Add email"
                  @click="toggleAddEmail"
                />
              </div>
            </q-card-section>

            <div class="q-pl-md text-subtitle1">
              SMS Settings (Overrides global SMS settings)
            </div>

            <q-card-section class="row">
              <div class="col-2 q-mb-sm">SMS recipients</div>
              <div class="col-4 q-mb-md">
                <q-list dense v-if="template.text_recipients.length !== 0">
                  <q-item
                    v-for="num in template.text_recipients"
                    :key="num"
                    dense
                  >
                    <q-item-section>
                      <q-item-label>{{ num }}</q-item-label>
                    </q-item-section>
                    <q-item-section side>
                      <q-icon
                        class="cursor-pointer"
                        name="delete"
                        color="red"
                        @click="removeSMSNumber(num)"
                      />
                    </q-item-section>
                  </q-item>
                </q-list>
                <q-list v-else>
                  <q-item-section>
                    <q-item-label>No recipients</q-item-label>
                  </q-item-section>
                </q-list>
              </div>
              <div class="col-3 q-mb-sm"></div>
              <div class="col-3 q-mb-sm">
                <q-btn
                  class="cursor-pointer"
                  size="sm"
                  icon="fas fa-plus"
                  color="secondary"
                  label="Add sms number"
                  @click="toggleAddSMSNumber"
                />
              </div>
            </q-card-section>
          </q-card>
        </q-step>

        <q-step :name="2" title="Alert Actions" icon="warning">
          <q-card flat>
            <div class="q-pl-md text-subtitle1">
              <span style="text-decoration: underline; cursor: help"
                >Alert Failure Settings
                <q-tooltip>
                  The selected script will run when an alert is triggered. This
                  script will run on any online agent.
                </q-tooltip>
              </span>
            </div>

            <q-card-section>
              <q-select
                class="q-mb-sm"
                label="Failure action"
                dense
                options-dense
                outlined
                clearable
                v-model="template.action"
                :options="scriptOptions"
                map-options
                emit-value
                @update:model-value="setScriptDefaults('failure')"
              >
                <template v-slot:option="scope">
                  <q-item
                    v-if="!scope.opt.category"
                    v-bind="scope.itemProps"
                    class="q-pl-lg"
                  >
                    <q-item-section>
                      <q-item-label v-html="scope.opt.label"></q-item-label>
                    </q-item-section>
                  </q-item>
                  <q-item-label
                    v-if="scope.opt.category"
                    v-bind="scope.itemProps"
                    header
                    class="q-pa-sm"
                    >{{ scope.opt.category }}</q-item-label
                  >
                </template>
              </q-select>

              <q-select
                class="q-mb-sm"
                dense
                label="Failure action arguments (press Enter after typing each argument)"
                filled
                v-model="template.action_args"
                use-input
                use-chips
                multiple
                hide-dropdown-icon
                input-debounce="0"
                new-value-mode="add"
              />

              <q-select
                class="q-mb-sm"
                dense
                label="Action en échec de variables d'environnement (Faire Entrée après chaque key=value pair)"
                filled
                v-model="template.action_env_vars"
                use-input
                use-chips
                multiple
                hide-dropdown-icon
                input-debounce="0"
                new-value-mode="add"
              />

              <q-input
                class="q-mb-sm"
                label="Action en échec au bout de (secondes)"
                outlined
                type="number"
                v-model.number="template.action_timeout"
                dense
                :rules="[
                  (val) => !!val || 'Failure action timeout is required',
                  (val) => val > 0 || 'Timeout must be greater than 0',
                  (val) => val <= 60 || 'Timeout must be 60 or less',
                ]"
              />
            </q-card-section>

            <div class="q-pl-md text-subtitle1">
              <span style="text-decoration: underline; cursor: help"
                >Alert Resolved Settings
                <q-tooltip>
                  The selected script will run when an alert is resolved. This
                  script will run on any online agent.
                </q-tooltip>
              </span>
            </div>

            <q-card-section>
              <q-select
                class="q-mb-sm"
                label="Action spécifique"
                dense
                options-dense
                outlined
                clearable
                v-model="template.resolved_action"
                :options="scriptOptions"
                map-options
                emit-value
                @update:model-value="setScriptDefaults('resolved')"
              >
                <template v-slot:option="scope">
                  <q-item
                    v-if="!scope.opt.category"
                    v-bind="scope.itemProps"
                    class="q-pl-lg"
                  >
                    <q-item-section>
                      <q-item-label v-html="scope.opt.label"></q-item-label>
                    </q-item-section>
                  </q-item>
                  <q-item-label
                    v-if="scope.opt.category"
                    v-bind="scope.itemProps"
                    header
                    class="q-pa-sm"
                    >{{ scope.opt.category }}</q-item-label
                  >
                </template>
              </q-select>

              <q-select
                class="q-mb-sm"
                dense
                label="Arguments d'action spécifique (Faire Entrée après chaque argument)"
                filled
                v-model="template.resolved_action_args"
                use-input
                use-chips
                multiple
                hide-dropdown-icon
                input-debounce="0"
                new-value-mode="add"
              />

              <q-select
                class="q-mb-sm"
                dense
                label="Action spécifiques de variables d'environnement (faire Entréé après chaque key=value pair)"
                filled
                v-model="template.resolved_action_env_vars"
                use-input
                use-chips
                multiple
                hide-dropdown-icon
                input-debounce="0"
                new-value-mode="add"
              />

              <q-input
                class="q-mb-sm"
                label="Action spécifique timeout (secondes)"
                outlined
                type="number"
                v-model.number="template.resolved_action_timeout"
                dense
                :rules="[
                  (val) => !!val || 'Resolved action timeout is required',
                  (val) => val > 0 || 'Timeout must be greater than 0',
                  (val) => val <= 60 || 'Timeout must be 60 or less',
                ]"
              />
            </q-card-section>

            <div class="q-pl-md text-subtitle1">
              <span style="text-decoration: underline; cursor: help"
                >Run actions only on
                <q-tooltip>
                  Le script sélectionné sera exécuté uniquement sur ces types d'alertes
                </q-tooltip>
              </span>
            </div>

            <q-card-section>
              <q-toggle
                v-model="template.agent_script_actions"
                label="Agents"
                color="green"
                left-label
              />

              <q-toggle
                v-model="template.check_script_actions"
                label="Contrôles"
                color="green"
                left-label
              />

              <q-toggle
                v-model="template.task_script_actions"
                label="Tâches"
                color="green"
                left-label
              />
            </q-card-section>
          </q-card>
        </q-step>

        <q-step :name="3" title="Paramètres des agents en retard" icon="devices">
          <q-card flat>
            <div class="q-pl-md text-subtitle1">
              <span style="text-decoration: underline; cursor: help"
                >Paramètres d'échec d'alerte
                <q-tooltip>
                  Sélectionner quelles notifications devront êtres envoyées quand un agent est
                  en retard. En étant activé, cela effacera les paramètres de notification de l'agent
                  et sera toujours notifié. Non configuré cela utilisera les paramètres de notification
                  configurés pour l'agent. Désactivé, cela effacera les paramètres de notification de l'agent et ne sera jamais notifié.
                </q-tooltip>
              </span>
            </div>
            <q-card-section>
              <q-toggle
                v-model="template.agent_always_email"
                label="Email"
                color="green"
                left-label
                toggle-indeterminate
              />
              <q-toggle
                v-model="template.agent_always_text"
                label="Texte"
                color="green"
                left-label
                toggle-indeterminate
              />
              <q-toggle
                v-model="template.agent_always_alert"
                label="Tableau de bord"
                color="green"
                left-label
                toggle-indeterminate
              />
            </q-card-section>
            <q-card-section>
              <q-input
                label="Alerter à nouveau si non résolu après (jours)"
                outlined
                type="number"
                v-model.number="template.agent_periodic_alert_days"
                dense
                :rules="[
                  (val) => val >= 0 || 'Periodic days must be 0 or greater',
                ]"
              />
            </q-card-section>

            <div class="q-pl-md text-subtitle1">
              <span style="text-decoration: underline; cursor: help"
                >Alert Resolved Settings
                <q-tooltip>
                  Sélectionner quelles notifications devrons être envoyées lorsqu'un agent en retard
                  est de retour en ligne.
                </q-tooltip>
              </span>
            </div>
            <q-card-section>
              <q-toggle
                v-model="template.agent_email_on_resolved"
                label="Email"
                color="green"
                left-label
              />
              <q-toggle
                v-model="template.agent_text_on_resolved"
                label="Texte"
                color="green"
                left-label
              />
            </q-card-section>
          </q-card>
        </q-step>

        <q-step :name="4" title="Paramètres de contrôle" icon="fas fa-check-double">
          <q-card flat>
            <div class="q-pl-md text-subtitle1">
              <span style="text-decoration: underline; cursor: help"
                >Alert Failure Settings
                <q-tooltip>
                  Sélectionner quelles notifications devront êtres envoyées quand un contrôle échoue. En étant activé, cela effacera les paramètres de notification de l'agent
                  et sera toujours notifié. Non configuré cela utilisera les paramètres de notification
                  configurés du contrôle. Désactivé, cela effacera les paramètres de notification du contrôle et ne sera jamais notifié.
                </q-tooltip>
              </span>
            </div>

            <q-card-section>
              <q-toggle
                v-model="template.check_always_email"
                label="Email"
                color="green"
                left-label
                toggle-indeterminate
              />
              <q-toggle
                v-model="template.check_always_text"
                label="Texte"
                color="green"
                left-label
                toggle-indeterminate
              />
              <q-toggle
                v-model="template.check_always_alert"
                label="Tableau de bord"
                color="green"
                left-label
                toggle-indeterminate
              />
            </q-card-section>

            <q-card-section>
              <q-select
                label="Envoyer un email uniquement en cas d'alerte severity"
                hint="Defaults to 'error' and 'warning'"
                v-model="template.check_email_alert_severity"
                outlined
                dense
                options-dense
                multiple
                use-chips
                emit-value
                map-options
                :options="severityOptions"
              />
            </q-card-section>

            <q-card-section>
              <q-select
                label="Envoyer un message texte uniquement en cas d'alerte severity"
                hint="Defaults to 'error' and 'warning'"
                v-model="template.check_text_alert_severity"
                outlined
                dense
                options-dense
                multiple
                use-chips
                emit-value
                map-options
                :options="severityOptions"
              />
            </q-card-section>

            <q-card-section>
              <q-select
                label="Afficher uniquement un alerte sur le tableau de bord en cas d'alerte on severity"
                hint="Defaults to 'error', 'warning', and 'info'"
                v-model="template.check_dashboard_alert_severity"
                outlined
                dense
                options-dense
                multiple
                use-chips
                emit-value
                map-options
                :options="severityOptions"
              />
            </q-card-section>

            <q-card-section>
              <q-input
                label="Envoyer à nouveau une alerte si non résolu après (jours)"
                outlined
                type="number"
                v-model.number="template.check_periodic_alert_days"
                dense
                :rules="[
                  (val) => val >= 0 || 'Periodic days must be 0 or greater',
                ]"
              />
            </q-card-section>

            <div class="q-pl-md text-subtitle1">
              <span style="text-decoration: underline; cursor: help"
                >Alert Resolved Settings
                <q-tooltip>
                  Sélectionner quelles notifications sont envoyées lorsqu'un contrôle échoué est résolu.
                </q-tooltip>
              </span>
            </div>
            <q-card-section>
              <q-toggle
                v-model="template.check_email_on_resolved"
                label="Email"
                color="green"
                left-label
              />
              <q-toggle
                v-model="template.check_text_on_resolved"
                label="Texte"
                color="green"
                left-label
              />
            </q-card-section>
          </q-card>
        </q-step>

        <q-step :name="5" title="Paramètres des tâches automatisées" icon="fas fa-tasks">
          <q-card flat>
            <div class="q-pl-md text-subtitle1">
              <span style="text-decoration: underline; cursor: help"
                >Alert Failure Settings
                <q-tooltip>Sélectionner quelles notifications devront êtres envoyées quand une tâche automatique échoue. En étant activé, cela effacera les paramètres de notification de la tâche
                  et sera toujours notifié. Non configuré cela utilisera les paramètres de notification
                  configurés de la tâche. Désactivé, cela effacera les paramètres de notification de la tâche et ne sera jamais notifié.                  
                </q-tooltip>
              </span>
            </div>

            <q-card-section>
              <q-toggle
                v-model="template.task_always_email"
                label="Email"
                color="green"
                left-label
                toggle-indeterminate
              />
              <q-toggle
                v-model="template.task_always_text"
                label="Texte"
                color="green"
                left-label
                toggle-indeterminate
              />
              <q-toggle
                v-model="template.task_always_alert"
                label="Tableau de bord"
                color="green"
                left-label
                toggle-indeterminate
              />
            </q-card-section>

            <q-card-section>
              <q-select
                label="Envoyer uniquement un email en cas d'alerte severity"
                hint="Defaults to 'error' and 'warning'"
                v-model="template.task_email_alert_severity"
                outlined
                dense
                options-dense
                multiple
                use-chips
                emit-value
                map-options
                :options="severityOptions"
              />
            </q-card-section>

            <q-card-section>
              <q-select
                label="Envoyer uniquement un message en cas d'alerte severity"
                hint="Defaults to 'error' and 'warning'"
                v-model="template.task_text_alert_severity"
                outlined
                dense
                options-dense
                multiple
                use-chips
                emit-value
                map-options
                :options="severityOptions"
              />
            </q-card-section>

            <q-card-section>
              <q-select
                label="Afficher uniquement une alerte sur le tableau de bord en cas de severity"
                hint="Defaults to 'error', 'warning', and 'info'"
                v-model="template.task_dashboard_alert_severity"
                outlined
                dense
                options-dense
                multiple
                use-chips
                emit-value
                map-options
                :options="severityOptions"
              />
            </q-card-section>

            <q-card-section>
              <q-input
                label="Envoyer à nouveau une alerte si non résolu après (jours)"
                outlined
                type="number"
                v-model.number="template.task_periodic_alert_days"
                dense
                :rules="[
                  (val) => val >= 0 || 'Periodic days must be 0 or greater',
                ]"
              />
            </q-card-section>

            <div class="q-pl-md text-subtitle1">
              <span style="text-decoration: underline; cursor: help"
                >Alert Resolved Settings
                <q-tooltip>
                  Select what notifications are sent when a failed task is
                  resolved.
                </q-tooltip>
              </span>
            </div>
            <q-card-section>
              <q-toggle
                v-model="template.task_email_on_resolved"
                label="Email"
                color="green"
                left-label
              />
              <q-toggle
                v-model="template.check_text_on_resolved"
                label="Texte"
                color="green"
                left-label
              />
            </q-card-section>
          </q-card>
        </q-step>
        <template v-slot:navigation>
          <q-stepper-navigation class="row">
            <q-btn
              v-if="step > 1"
              flat
              color="primary"
              @click="$refs.stepper.previous()"
              label="Back"
              class="q-mr-xs"
            />
            <q-btn
              v-if="step < 5"
              @click="$refs.stepper.next()"
              color="primary"
              label="Next"
            />
            <q-space />
            <q-btn @click="onSubmit" color="primary" label="Submit" />
          </q-stepper-navigation>
        </template>
      </q-stepper>
    </q-card>
  </q-dialog>
</template>

<script>
import mixins from "@/mixins/mixins";
import { mapGetters } from "vuex";

export default {
  name: "AlertTemplateForm",
  emits: ["hide", "ok", "cancel"],
  mixins: [mixins],
  props: { alertTemplate: Object },
  data() {
    return {
      step: 1,
      template: {
        name: "",
        is_active: true,
        action: null,
        action_args: [],
        action_env_vars: [],
        action_timeout: 15,
        resolved_action: null,
        resolved_action_args: [],
        resolved_action_env_vars: [],
        resolved_action_timeout: 15,
        email_recipients: [],
        email_from: "",
        text_recipients: [],
        agent_email_on_resolved: false,
        agent_text_on_resolved: false,
        agent_always_email: null,
        agent_always_text: null,
        agent_always_alert: null,
        agent_periodic_alert_days: 0,
        agent_script_actions: true,
        check_email_alert_severity: [],
        check_text_alert_severity: [],
        check_dashboard_alert_severity: [],
        check_email_on_resolved: false,
        check_text_on_resolved: false,
        check_always_email: null,
        check_always_text: null,
        check_always_alert: null,
        check_periodic_alert_days: 0,
        check_script_actions: true,
        task_email_alert_severity: [],
        task_text_alert_severity: [],
        task_dashboard_alert_severity: [],
        task_email_on_resolved: false,
        task_text_on_resolved: false,
        task_always_email: null,
        task_always_text: null,
        task_always_alert: null,
        task_periodic_alert_days: 0,
        task_script_actions: true,
      },
      scriptOptions: [],
      severityOptions: [
        { label: "Error", value: "error" },
        { label: "Warning", value: "warning" },
        { label: "Informational", value: "info" },
      ],
      thumbStyle: {
        right: "2px",
        borderRadius: "5px",
        backgroundColor: "#027be3",
        width: "5px",
        opacity: 0.75,
      },
    };
  },
  computed: {
    ...mapGetters(["showCommunityScripts"]),
    title() {
      return this.editing ? "Edit Alert Template" : "Add Alert Template";
    },
    editing() {
      return !!this.alertTemplate;
    },
  },
  methods: {
    setScriptDefaults(type) {
      if (type === "failure") {
        const script = this.scriptOptions.find(
          (i) => i.value === this.template.action
        );
        this.template.action_args = script.args;
        this.template.action_env_vars = script.env_vars;
      } else if (type === "resolved") {
        const script = this.scriptOptions.find(
          (i) => i.value === this.template.resolved_action
        );
        this.template.resolved_action_args = script.args;
        this.template.resolved_action_env_vars = script.env_vars;
      }
    },
    toggleAddEmail() {
      this.$q
        .dialog({
          title: "Ajouter un email",
          prompt: {
            model: "",
            isValid: (val) => this.isValidEmail(val),
            type: "email",
          },
          cancel: true,
          ok: { label: "Ajouter", color: "primary" },
          persistent: false,
        })
        .onOk((data) => {
          this.template.email_recipients.push(data);
        });
    },
    toggleAddSMSNumber() {
      this.$q
        .dialog({
          title: "Ajouter un chiffre",
          message:
            "Use E.164 format: must have the <b>+</b> symbol and <span class='text-red'>country code</span>, followed by the <span class='text-green'>phone number</span> e.g. <b>+<span class='text-red'>1</span><span class='text-green'>2131231234</span></b>",
          prompt: {
            model: "",
          },
          html: true,
          cancel: true,
          ok: { label: "Ajouter", color: "primary" },
          persistent: false,
        })
        .onOk((data) => {
          this.template.text_recipients.push(data);
        });
    },
    removeEmail(email) {
      const removed = this.template.email_recipients.filter((k) => k !== email);
      this.template.email_recipients = removed;
    },
    removeSMSNumber(num) {
      const removed = this.template.text_recipients.filter((k) => k !== num);
      this.template.text_recipients = removed;
    },
    onSubmit() {
      if (!this.template.name) {
        this.notifyError("Name needs to be set");
        return;
      }

      this.$q.loading.show();

      if (this.editing) {
        this.$axios
          .put(`alerts/templates/${this.template.id}/`, this.template)
          .then(() => {
            this.$q.loading.hide();
            this.onOk();
            this.notifySuccess("Alert Template edited!");
          })
          .catch(() => {
            this.$q.loading.hide();
          });
      } else {
        this.$axios
          .post("alerts/templates/", this.template)
          .then(() => {
            this.$q.loading.hide();
            this.onOk();
            this.notifySuccess("Alert Template was added!");
          })
          .catch(() => {
            this.$q.loading.hide();
          });
      }
    },
    show() {
      this.$refs.dialog.show();
    },
    hide() {
      this.$refs.dialog.hide();
    },
    onHide() {
      this.$emit("hide");
    },
    onOk() {
      this.$emit("ok");
      this.hide();
    },
  },
  mounted() {
    this.getScriptOptions(this.showCommunityScripts).then(
      (options) => (this.scriptOptions = Object.freeze(options))
    );
    // Copy alertTemplate prop locally
    if (this.editing) Object.assign(this.template, this.alertTemplate);
  },
};
</script>
