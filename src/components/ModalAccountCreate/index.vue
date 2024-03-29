<template>
  <div class="flex justify-between">
    <h1 class="font-black text-4xl text-gray-800">Crie uma conta</h1>
    <button class="text-4xl text-gray-600 focus:outline-none" @click="close">
      &times;
    </button>
  </div>

  <div class="mt-16">
    <form @submit.prevent="handleSubmit">
      <label class="block">
        <span class="text-lg font-gray-800 font-medium">Nome</span>
        <input
          class="block w-full px-4 py-3 mt-1 text-lg bg-gray-100 border-2 border-transparent rounded"
          :class="{ 'border-brand-danger': !!state.name.errorMessage }"
          placeholder="Jane Doe"
          v-model="state.name.value"
        />
        <span
          v-if="!!state.name.errorMessage"
          class="block font-medium text-brand-danger"
        >
          {{ state.name.errorMessage }}
        </span>
      </label>

      <label class="block mt-9">
        <span class="text-lg font-gray-800 font-medium">E-mail</span>
        <input
          type="email"
          class="block w-full px-4 py-3 mt-1 text-lg bg-gray-100 border-2 border-transparent rounded"
          :class="{ 'border-brand-danger': !!state.email.errorMessage }"
          placeholder="jane.doe@email.com"
          v-model="state.email.value"
        />
        <span
          v-if="!!state.email.errorMessage"
          class="block font-medium text-brand-danger"
        >
          {{ state.email.errorMessage }}
        </span>
      </label>

      <label class="block mt-9">
        <span class="text-lg font-gray-800 font-medium">Senha</span>
        <input
          type="password"
          class="block w-full px-4 py-3 mt-1 text-lg bg-gray-100 border-2 border-transparent rounded"
          :class="{ 'border-brand-danger': !!state.password.errorMessage }"
          placeholder="******"
          v-model="state.password.value"
        />
        <span
          v-if="!!state.password.errorMessage"
          class="block font-medium text-brand-danger"
        >
          {{ state.password.errorMessage }}
        </span>
      </label>

      <button
        :disabled="state.isLoading"
        type="submit"
        class="px-8 py-3 mt-10 text-2xl text-white bg-brand-main rounded-full focus:outline-none transition-all duration-150"
        :class="{ 'opacity-50': state.isLoading }"
      >
        <Icon v-if="state.isLoading" name="loading" class="animate-spin" />
        <span v-else>Entrar</span>
      </button>
    </form>
  </div>
</template>

<script>
import { reactive } from 'vue'

// Hooks
import useModal from '../../hooks/useModal'
import { useField } from 'vee-validate'
import { useRouter } from 'vue-router'

// Validations
import { custom } from '../../utils/validators'

// Services
import services from '../../services'

// Componenets
import Icon from '../../components/Icon'

export default {
  name: 'ModalAccountCreate',

  components: {
    Icon
  },

  setup() {
    const router = useRouter()
    const modal = useModal()

    const { value: nameValue, errorMessage: nameErrorMessage } = useField(
      'name',
      custom.emptyAndLenght3
    )

    const { value: emailValue, errorMessage: emailErrorMessage } = useField(
      'email',
      custom.emptyAndEmail
    )

    const {
      value: passwordValue,
      errorMessage: passwordErrorMessage
    } = useField('password', custom.emptyAndLenght6)

    const state = reactive({
      hasError: false,
      isLoading: false,
      name: {
        value: nameValue,
        errorMessage: nameErrorMessage
      },
      email: {
        value: emailValue,
        errorMessage: emailErrorMessage
      },
      password: {
        value: passwordValue,
        errorMessage: passwordErrorMessage
      }
    })

    async function login({ email, password }) {
      const { data, errors } = await services.auth.login({ email, password })

      if (!errors) {
        localStorage.setItem('token', data.token)
        router.push({ name: 'Feedbacks' })
        modal.close()
      }

      state.isLoading = false
    }

    async function handleSubmit() {
      try {
        state.isLoading = true
        const { errors } = await services.auth.register({
          name: state.name.value,
          email: state.email.value,
          password: state.password.value
        })

        if (!errors) {
          await login({
            email: state.email.value,
            password: state.password.value
          })

          return
        }
      } catch (error) {
        state.isLoading = false
        state.hasError = !!error
      }
    }

    return { state, close: modal.close, handleSubmit }
  }
}
</script>
