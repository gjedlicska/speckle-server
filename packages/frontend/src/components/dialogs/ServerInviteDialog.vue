<template>
  <div>
    <v-dialog v-model="showDialog" max-width="400">
      <v-card>
        <v-card-title>Send a server invite</v-card-title>
        <v-alert v-model="showError" dismissible type="error" :class="`${success ? 'mb-0' : ''}`">
          {{ error }}
        </v-alert>
        <v-alert v-model="success" timeout="3000" dismissible type="success">
          Great! An invite link has been sent.
        </v-alert>
        <v-form ref="form" v-model="valid" class="px-2" @submit.prevent="sendInvite">
          <v-card-text class="pb-0 mb-0">
            We will send an invite link for this server to the email below. You can also add a
            personal message if you want to.
          </v-card-text>
          <v-card-text class="pt-0 mt-0">
            <v-text-field
              v-model="email"
              :rules="validation.emailRules"
              label="email"
            ></v-text-field>
            <v-text-field
              v-model="message"
              :rules="validation.messageRules"
              label="message"
            ></v-text-field>
            <v-card-actions>
              <v-btn block color="primary" type="submit">Send invite</v-btn>
            </v-card-actions>
          </v-card-text>
        </v-form>
      </v-card>
    </v-dialog>
  </div>
</template>
<script>
import gql from 'graphql-tag'
import DOMPurify from 'dompurify'

export default {
  name: 'ServerInviteDialog',
  data() {
    return {
      showDialog: false,
      email: null,
      message: 'Hey, join this Speckle Server!',
      valid: false,
      error: null,
      showError: false,
      success: false,
      validation: {
        emailRules: [
          (v) => !!v || 'E-mail is required',
          (v) => /.+@.+\..+/.test(v) || 'E-mail must be valid'
        ],
        messageRules: [
          (v) => {
            if (v.length >= 1024) return 'Message too long!'
            return true
          },
          (v) => {
            let pure = DOMPurify.sanitize(v)
            if (pure !== v) return 'No crazy hacks please.'
            else return true
          }
        ]
      }
    }
  },
  watch: {
    showDialog() {
      this.clear()
      this.email = null
      this.message = 'Hey, join this Speckle Server!'
    }
  },
  methods: {
    show() {
      this.showDialog = true
    },
    clear() {
      this.error = null
      this.showError = false
      this.success = false
      if (this.$refs.form) this.$refs.form.resetValidation()
    },
    async sendInvite() {
      if (!this.$refs.form.validate()) return

      this.$matomo && this.$matomo.trackPageView('invite/create')
      this.$matomo && this.$matomo.trackEvent('invite', 'server')
      try {
        await this.$apollo.mutate({
          mutation: gql`
            mutation($input: ServerInviteCreateInput!) {
              serverInviteCreate(input: $input)
            }
          `,
          variables: {
            input: {
              email: this.email,
              message: this.message
            }
          }
        })

        this.clear()
        this.success = true
      } catch (e) {
        this.clear()
        this.showError = true
        this.error = e.message
      }
    }
  }
}
</script>
