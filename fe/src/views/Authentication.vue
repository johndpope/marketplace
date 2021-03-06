<template>
  <div class="auth">
    <div class="auth__form">
      <h1>{{message}}</h1>
      <form @submit.prevent="handleSubmit">
        <div class="auth__form__group">
          <label for="email">Correo Electrónico</label>
          <input type="email" name="email" id="email" v-model="user.email" required>
        </div>
        <div class="auth__form__group">
          <label for="password">Contraseña</label>
          <input type="password" name="password" id="password" v-model="user.password" required>
        </div>
        <div v-if="isRegisterMode" class="auth__form__group">
          <label for="rePassword">Ingrese contraseña nuevamente</label>
          <input type="password" name="rePassword" id="rePassword" v-model="user.rePassword" required>
        </div>
        <button
            class="mktpl-btn__primary__fullwidth"
            type="submit" v-if="!isRegisterMode">
          Iniciar Sesión
        </button>
        <button
            class="mktpl-btn__success__fullwidth"
            type="button" v-if="!isRegisterMode"
            @click="isRegisterMode=true">
          Registrar
        </button>
        <button
            class="mktpl-btn__success__fullwidth"
            type="submit" v-if="isRegisterMode">
          Crear Cuenta
        </button>
      </form>
    </div>
  </div>
</template>

<script lang="ts">
  import {Component, Vue} from 'vue-property-decorator';
  import {AuthService} from '@/services/AuthService';
  import {IUserModel} from '@/entities/IUserModel';
  import Swal from 'sweetalert2';
  import {RawLocation} from 'vue-router';
  import {LOGIN_ERROR, LOGIN_SUCCESS, REGISTER_ERROR, REGISTER_SUCCESS} from '@/messages/Dialogs';

  @Component({
  components: {},
  data: () => ({
    user: {email: '', password: '', rePassword: ''},
  }),
})
export default class Authentication extends Vue {
  private isRegisterMode: boolean = false;
  private user?: IUserModel = {email: '', password: ''};
  private fromUrl?: RawLocation = '';
  private toUrl?: RawLocation = '';
  private message = 'Debe iniciar sesión.';

  private created() {
    this.fromUrl = (this.$route.query as any).from;
    this.toUrl = (this.$route.query as any).to;
  }

  private handleSubmit(e: MouseEvent) {
    e.preventDefault();
    const authService = new AuthService();
    if (this.user) {
      if (this.isRegisterMode) {
        this.register(authService, this.user);
      } else {
        this.login(authService, this.user);
      }
    }
  }

  private async login(authService: AuthService, user: IUserModel): Promise<any> {
    try {
      const result = await authService.login(user);
      if (!result) {
        Swal.fire(LOGIN_ERROR);
      } else {
        Swal.fire(LOGIN_SUCCESS)
          .then(() => {
            this.$router.push(this.toUrl || '');
          });
      }
    } catch (e) {
      const errorDialog = LOGIN_ERROR;
      errorDialog.text += e;
      Swal.fire(errorDialog);
    }
  }

  private async register(authService: AuthService, user: IUserModel) {
    try {
      const result = await authService.register(user);
      if (result) {
        Swal.fire(REGISTER_SUCCESS).then(() => {
          this.isRegisterMode = false;
        });
      }
    } catch (error) {
      const errorDialog = REGISTER_ERROR;
      errorDialog.text += error;
      Swal.fire(errorDialog);
    }
  }
}
</script>

<style lang="scss" scoped>
  .auth {
    width: 100%;
    display: flex;
    flex-flow: row wrap;
    background-color: #d3d3d361;

    &__form {
      width: 100%;
      text-align: start;
      margin: 0.5rem;
      padding: 0.5rem;
      background-color: white;
      box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2);
      transition: 0.3s;

      &__group {
        margin-bottom: 1rem;
      }
    }
  }
</style>
