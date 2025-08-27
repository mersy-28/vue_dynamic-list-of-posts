<template>
  <form @submit.prevent="handleSubmit">
    <div class="field">
      <label class="label">Name</label>
      <div class="control">
        <input
          class="input"
          v-model="form.authorName"
          :class="{ 'is-danger': errors.authorName }"
          type="text"
          placeholder="Your name"
          required
        />
      </div>
      <p v-if="errors.authorName" class="help is-danger">
        {{ errors.authorName }}
      </p>
    </div>
    <div class="field">
      <label class="label">Email</label>
      <div class="control">
        <input
          class="input"
          v-model="form.authorEmail"
          :class="{ 'is-danger': errors.authorEmail }"
          type="email"
          placeholder="Your email"
          required
        />
      </div>
      <p v-if="errors.authorEmail" class="help is-danger">
        {{ errors.authorEmail }}
      </p>
    </div>
    <div class="field">
      <label class="label">Comment</label>
      <div class="control">
        <textarea
          class="textarea"
          v-model="form.text"
          :class="{ 'is-danger': errors.text }"
          placeholder="Write your comment"
          required
        ></textarea>
      </div>
      <p v-if="errors.text" class="help is-danger">{{ errors.text }}</p>
    </div>
    <div class="field is-grouped">
      <div class="control">
        <button
          class="button is-link"
          :class="{ 'is-loading': loading }"
          type="submit"
        >
          Submit
        </button>
      </div>
      <div class="control">
        <button class="button is-light" type="button" @click="handleClear">
          Clear
        </button>
      </div>
      <div class="control">
        <button class="button is-light" type="button" @click="$emit('cancel')">
          Cancel
        </button>
      </div>
    </div>
    <p v-if="error" class="help is-danger">{{ error }}</p>
  </form>
</template>

<script>
import { addComment } from "./api.js";
export default {
  name: "CommentForm",
  props: {
    postId: Number,
  },
  data() {
    return {
      form: {
        authorName: '',
        authorEmail: '',
        text: '',
      },
      errors: {},
      loading: false,
      error: '',
    };
  },
  methods: {
    validate() {
      this.errors = {};
      if (!this.form.authorName) this.errors.authorName = 'Name is required';
      if (!this.form.authorEmail) this.errors.authorEmail = 'Email is required';
      else if (!/^[^@\s]+@[^@\s]+\.[^@\s]+$/.test(this.form.authorEmail)) this.errors.authorEmail = 'Invalid email';
      if (!this.form.text) this.errors.text = 'Comment is required';
      return Object.keys(this.errors).length === 0;
      },
      async handleSubmit() {
        if (!this.validate()) return;
        this.loading = true;
        this.error = '';
        try {
          const newComment = await addComment(this.postId, { ...this.form });
          this.$emit('submitted', newComment);
          // Keep name/email, clear only comment text
          this.form.text = '';
          this.errors.text = '';
        } catch (err) {
          this.error = err.message || 'Failed to add comment';
        } finally {
          this.loading = false;
        }
      },
      handleClear() {
        this.form.authorName = '';
        this.form.authorEmail = '';
        this.form.text = '';
        this.errors = {};
        this.error = '';
      },
    },
    watch: {
      'form.authorName'(val) { if (this.errors.authorName) this.errors.authorName = ''; },
      'form.authorEmail'(val) { if (this.errors.authorEmail) this.errors.authorEmail = ''; },
      'form.text'(val) { if (this.errors.text) this.errors.text = ''; },
    },
  };
    },
    "form.text"(val) {
      if (this.errors.text) this.errors.text = "";
    },
  },
};
</script>
