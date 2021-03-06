<template>
  <div />
</template>

<script lang="ts">
import Vue from 'vue';

import { Declaration, Rule } from 'postcss';
import { StylebotEditingMode, StylebotPlacement } from '@stylebot/types';

export default Vue.extend({
  name: 'TheKeyboardShortcuts',

  computed: {
    visible(): boolean {
      return this.$store.state.visible;
    },
    inspecting(): boolean {
      return this.$store.state.inspecting;
    },
    mode(): StylebotEditingMode {
      return this.$store.state.options.mode;
    },
    position(): StylebotPlacement {
      return this.$store.state.position;
    },
    activeSelector(): string {
      return this.$store.state.activeSelector;
    },
    activeRule(): Rule {
      return this.$store.getters.activeRule;
    },
    help(): boolean {
      return this.$store.state.help;
    },
  },

  watch: {
    visible(newValue: boolean): void {
      if (newValue) {
        this.attachStylebotShortcuts();
      } else {
        this.detachStylebotShortcuts();
      }
    },
  },

  methods: {
    attachStylebotShortcuts(): void {
      document.addEventListener('keydown', this.handleStylebotShortcut);
    },

    detachStylebotShortcuts(): void {
      document.removeEventListener('keydown', this.handleStylebotShortcut);
    },

    toggleInspect(): void {
      if (this.mode === 'basic') {
        this.$store.commit('setInspecting', !this.inspecting);
      }
    },

    togglePosition(): void {
      this.$store.commit(
        'setPosition',
        this.position === 'left' ? 'right' : 'left'
      );
    },

    toggleVisibilityOfActiveSelector(): void {
      if (this.activeSelector) {
        let value = '';

        if (this.activeRule) {
          this.activeRule.clone().walkDecls('display', (decl: Declaration) => {
            value = decl.value;
          });
        }

        this.$store.dispatch('applyDeclaration', {
          property: 'display',
          value: value === 'none' ? '' : 'none',
        });
      }
    },

    toggleHelp(): void {
      this.$store.commit('setInspecting', false);
      this.$store.commit('setHelp', !this.help);
    },

    handleEscape(): void {
      if (this.help) {
        this.$store.commit('setHelp', false);
        return;
      }

      this.$store.dispatch('closeStylebot');
    },

    handleStylebotShortcut(event: KeyboardEvent): void {
      const target = event.composedPath()[0] as HTMLElement;
      if (target.tagName.toLowerCase() === 'input') {
        return;
      }

      if (event.metaKey || event.altKey) {
        return;
      }

      // i - Toggle inspect
      if (event.keyCode === 73) {
        event.preventDefault();
        event.stopPropagation();

        this.toggleInspect();
      }

      // p - Toggle editor position
      if (event.keyCode === 80) {
        event.preventDefault();
        event.stopPropagation();

        this.togglePosition();
      }

      // h - Toggle visibility css of selected element(s)
      if (event.keyCode === 72) {
        event.preventDefault();
        event.stopPropagation();

        this.toggleVisibilityOfActiveSelector();
      }

      // b - switch to basic editor
      if (event.keyCode === 66) {
        event.preventDefault();
        event.stopPropagation();

        this.$store.dispatch('setMode', 'basic');
      }

      // c - switch to code editor
      if (event.keyCode === 67) {
        event.preventDefault();
        event.stopPropagation();

        this.$store.dispatch('setMode', 'code');
      }

      // m - switch to magic editor
      if (event.keyCode === 77) {
        event.preventDefault();
        event.stopPropagation();

        this.$store.dispatch('setMode', 'magic');
      }

      // ? - show shortcut help
      if (event.keyCode === 191 && event.shiftKey) {
        event.preventDefault();
        event.stopPropagation();

        this.toggleHelp();
      }

      // esc - if help is visible, close help. else, close stylebot
      if (event.keyCode === 27) {
        event.preventDefault();
        event.stopPropagation();

        this.handleEscape();
      }
    },
  },
});
</script>
