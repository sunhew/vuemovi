<template>
  <div class="email-container">
    <a :href="mailToLink" class="liarusen@gmail.com">liarusen@gmail.com</a>
  </div>
</template>

<script>
export default {
  data() {
    return {
      email: 'example@example.com',
      subject: 'Hello',
      body: 'This is a test email.'
    }
  },
  computed: {
    mailToLink() {
      return `mailto:${this.email}?subject=${encodeURIComponent(this.subject)}&body=${encodeURIComponent(this.body)}`
    }
  }
}
</script>

<style scoped>
.email-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 50px;
  background-color: var(--pointBg);
  color: var(--white);
  font-size: 1.5rem;
}
</style>
