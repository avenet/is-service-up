<template>
  <div>
      <div class="loading" v-if="!services || services.length == 0">
        <img src="/images/loading.gif" alt="loading.." title="loading.." />
        <div>Loading service status..</div>
      </div>
      <div class="services-container one-column" id="services" v-if="services && services.length > 0">
        <service v-for="service in sortedServices" :service="service" :key="service.name"></service>
      </div>
      <div class="last-update" v-if="last_update != null">
        Last update: <span class="last-update-seconds">{{last_update}}</span> seconds ago
      </div>
  </div>
</template>

<script>
import * as config from '../config';
import {spawnNotification} from '../utils/notifications';
import Service from './Service';

function notifyStatusChange(serviceName, serviceIcon, oldStatus, newStatus) {
  var options = {
    icon: serviceIcon
  };
  var statusDesc = config.STATUS_DESCRIPTION[newStatus];
  var msg = serviceName + '\'s new status is "' + statusDesc + '"';
  spawnNotification(msg, options);
}

export default {
  name: 'services',
  components: {
    Service,
  },

  data: function() {
    return  {
      services: [],
      last_update: null,
    };
  },

  created: function _ready(){
    this.fetchServices();
    setInterval(this.fetchServices.bind(this), config.SERVICES_REFRESH_INTERVAL * 1000);
    setInterval(this.increaseLastUpdate.bind(this), 1000);
  },

  methods: {

    fetchServices: function _fetchServices(){
      var self = this;
      $.get(config.API_HOST + '/status', function(data) {
        var services = data.data.services;
        var prevServicesMap = {};
        $.each(self.services, function(i, service) {
          prevServicesMap[service.name] = service;
        });
        $.each(services, function(i, service) {
          if (prevServicesMap
              && prevServicesMap[service.name]
              && service.status != prevServicesMap[service.name].status) {
            var prevStatus = prevServicesMap[service.name].status;
            notifyStatusChange(service.name, service.icon_url, prevStatus, service.status);
          }
        });

        self.services = services;
        self.last_update = 0;
      });
    },

    increaseLastUpdate: function() {
      if (this.last_update === null) {
        return;
      }
      this.last_update += 1;
    },

  },

  computed: {
    sortedServices: function () {
      var clone = this.services.slice(0);
      clone.sort((a,b) => (a.name.toLowerCase() > b.name.toLowerCase()) ? 1 : -1);
      return clone;
    }
  }
}
</script>
