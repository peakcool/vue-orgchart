<template>
  <div id="app">
    <org-chart :datasource="ds" @node-click="selectNode" :zoom="true" :pan="true" :active-node="activeNode" :options="options">
    </org-chart>
  </div>
</template>

<script>
import OrgChart from './components/OrganizationChartContainer.vue'

export default {
  name: 'app',
  components: {
    OrgChart
  },
  data () {
    return {
      activeNode: {
        id: 3
      },
      options: {
        direction: 'l2r',
        showGrid: false
      },
      ds: {
        'id': '1',
        'name': 'Lao Lao',
        'subtitle': 'general manager',
        'children': [
          { 'id': '2', 'name': 'Bo Miao', 'subtitle': 'department manager' },
          { 'id': '3', 'name': 'Su Miao', 'subtitle': 'department manager',
            'children': [
              { 'id': '4', 'name': 'Tie Hua', 'subtitle': 'senior engineer' },
              { 'id': '5', 'name': 'Hei Hei', 'subtitle': 'senior engineer',
                'children': [
                  { 'id': '6', 'name': 'Pang Pang', 'subtitle': 'engineer' },
                  { 'id': '7', 'name': 'Xiang Xiang', 'subtitle': 'UE engineer' }
                ]
               }
             ]
           },
          { 'id': '8', 'name': 'Hong Miao', 'subtitle': 'department manager' },
          { 'id': '9', 'name': 'Chun Miao', 'subtitle': 'department manager' }
        ]
      }
    }
  },
  methods: {
    selectNode (nodeData) {
      alert('node ' + nodeData.name + ' is selected')
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
