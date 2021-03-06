<template>
  <transition-group
    name="list" tag="div"
    class="apos-notifications" :class="themeClass"
  >
    <AposNotification
      v-for="notification in notifications"
      :key="notification._id"
      :label="notification.message"
      :type="notification.type"
      :icon="notification.icon"
      :id="notification._id"
      :dismiss="notification.dismiss"
      :buttons="notification.buttons"
      @close="dismiss"
    />
  </transition-group>
</template>

<script>
import AposThemeMixin from 'Modules/@apostrophecms/ui/mixins/AposThemeMixin';
export default {
  name: 'TheAposNotifications',
  mixins: [ AposThemeMixin ],
  data () {
    return {
      notifications: [],
      dismissed: []
    };
  },
  async mounted() {
    apos.notify = async function(message, options) {
      const strings = [];
      let i = 1;
      let index = 0;
      while (true) {
        index = message.indexOf('%s', index);
        if (index === -1) {
          break;
        }
        // Don't match the same one over and over
        index += 2;
        if ((i >= arguments.length) || ((typeof (arguments[i]) === 'object'))) {
          throw new Error('Bad notification call: number of %s placeholders does not match number of string arguments after message');
        }
        strings.push(arguments[i++]);
      }
      if ((i === (arguments.length - 1)) && (typeof (arguments[i]) === 'object')) {
        options = arguments[i++];
      } else {
        options = {};
      }

      if (i !== arguments.length) {
        throw new Error('Bad notification call: number of %s placeholders does not match number of string arguments after message');
      }

      if (options.dismiss === true) {
        options.dismiss = 5;
      }

      // Send it to the server, which will send it back to us via
      // the same long polling mechanism that allows it to reach
      // other tabs, and allows server-sent notifications to
      // reach us

      await apos.http.post(apos.notification.action, {
        body: {
          message,
          strings,
          type: options.type,
          icon: options.icon,
          dismiss: options.dismiss,
          buttons: options.buttons
        }
      });
    };

    await this.poll();
  },
  methods: {
    async dismiss(notificationId) {
      await apos.http.patch(`${apos.notification.action}/${notificationId}`, {
        body: {
          dismissed: true
        }
      });
      this.notifications = this.notifications.reduce((acc, cur) => {
        if (cur._id !== notificationId) {
          acc.push(cur);
        }
        return acc;
      }, []);
    },
    async poll() {
      try {
        if (document.visibilityState === 'hidden') {
          // Wait for tab to become visible
          setTimeout(this.poll, 5000);
        } else {
          const allNotifications = [ ...this.notifications, ...this.dismissed ];
          const latestTimestamp = allNotifications
            .map(notification => notification.updatedAt)
            .sort()
            .reverse()[0];

          const seenIds = allNotifications
            .filter(notification => notification.updatedAt === latestTimestamp)
            .map(notification => notification._id);

          const { notifications, dismissed } = await apos.http.get(apos.notification.action, {
            ...(latestTimestamp && {
              qs: {
                modifiedOnOrSince: latestTimestamp,
                seenIds
              }
            })
          });

          this.notifications = [ ...this.notifications, ...(notifications || []) ];
          this.dismissed = [ ...this.dismissed, ...(dismissed || []) ];

          if (dismissed.length) {
            this.notifications = this.notifications.filter(notification => {
              return !dismissed.some(element => notification._id === element._id);
            });
          }
          // Long polling, we should reconnect promptly, the server
          // is responsible for keeping that request open for a reasonable
          // amount of time if there are no new messages, not us
          setTimeout(this.poll, 50);
        }
      } catch (err) {
        console.error(err);
        setTimeout(this.poll, 5000);
      }
    }
  }
};
</script>

<style lang="scss" scoped>
  .apos-notifications {
    z-index: $z-index-notifications;
    pointer-events: none;
    position: fixed;
    bottom: 20px;
    display: flex;
    width: 100%;
    flex-direction: column;
    align-items: center;
  }

  .list-enter-active,
  .list-leave-active {
    @include apos-transition();
  }

  .list-enter,
  .list-leave-to {
    opacity: 0;
    transform: translateY(5px);
  }

</style>
