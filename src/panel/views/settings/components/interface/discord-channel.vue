<template>
  <div>
    <div class="d-flex">
      <div class="input-group-prepend">
        <span class="input-group-text">
          <template v-if="typeof translatedTitle === 'string'">{{ translatedTitle }}</template>
          <template v-else>
            {{ translatedTitle.title }}
            <small class="text-info" data-toggle="tooltip" data-html="true" :title="translatedTitle.help">[?]</small>
          </template>
        </span>
      </div>
      <b-form-select v-if="typeof currentValue === 'string'" v-model="currentValue" :options="channels"></b-form-select>
    </div>
    <template v-if="typeof currentValue === 'object'">
      <div class="d-flex" v-for="key of Object.keys(currentValue)" :key="key" >
        <div class="input-group-prepend">
          <span class="input-group-text">
            {{ key }}
          </span>
        </div>
        <b-form-select v-model="currentValue[key]" :options="channels"></b-form-select>
      </div>
    </template>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Prop, Watch } from 'vue-property-decorator';
import { getSocket } from 'src/panel/helpers/socket';
import { announceTypes } from 'src/bot/commons';

type Channel = { text: string, value: string };

@Component({})
export default class discordChannel extends Vue {
  @Prop() readonly value!: string | { [key in typeof announceTypes[number]]: string };
  @Prop() readonly title!: string;

  socket = getSocket('/integrations/discord')
  channels: Channel[] = []

  currentValue = this.value;
  translatedTitle = this.translate(this.title);

  mounted() {
    this.socket.emit('discord::getChannels', (err: string | null, channels: Channel[]) => {
      console.groupCollapsed('discord::getChannels')
      console.log({channels});
      console.groupEnd();
      if (err) {
        return console.error(err);
      }

      // find channel in channels on current or unset current
      if (typeof this.currentValue !== 'object') {
        if (!channels.find(o => String(o.value) === String(this.currentValue))) {
          this.currentValue = '';
        }
      } else {
        for (const key of Object.keys(this.currentValue) as Writeable<typeof announceTypes>) {
          if (!channels.find(o => {
            return typeof this.currentValue === 'object' && String(o.value) === String(this.currentValue[key]);
          })) {
            this.currentValue[key] = '';
          }
        }
      }
      this.channels = [{ value: '', text: `-- ${this.translate('integrations.discord.settings.noChannelSelected')} --` }, ...channels];
    });
  }

  @Watch('currentValue', { deep: true })
  onChange() {
    this.$emit('update', { value: this.currentValue });
  }
};
</script>
