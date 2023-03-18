<template>
  <div id="app" class="dark-mode">
    <div class="nodes-container">
      <h2>Swiss Staking Load Balancer</h2>
      <div class="filter-container">
        <button v-for="path in filteredPaths" :key="path" @click="setPathFilter(path)">
          {{ path }}
        </button>
      </div>
      <table>
        <thead>
         {{ path }}.swiss-staking.ch
        </thead>
        <tbody>
          <tr v-for="node in filteredNodes" :key="node.name">
            <td>{{ node.url }}</td>
            <td>
              <span class="status-circle" :class="node.status"></span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      repositories: [
        {
          username: 'creeea',
          repository: 'web3-load-balancer-configuration',
          path: 'juno/rpc.json',
        },
        {
          username: 'creeea',
          repository: 'web3-load-balancer-configuration',
          path: 'juno/lcd.json',
        },
        {
          username: 'creeea',
          repository: 'web3-load-balancer-configuration',
          path: 'evmos/lcd.json',
        },
        // Add more repositories here...
      ],
      nodes: [],
      pathFilter: 'juno/lcd.json',
    };
  },
  created() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      this.repositories.forEach(repository => {
        const url = `https://api.github.com/repos/${repository.username}/${repository.repository}/contents/${repository.path}`;
        fetch(url)
          .then(response => response.json())
          .then(data => {
            const decodedData = JSON.parse(atob(data.content));
            
            const nodesWithPaths = decodedData.map(node => ({ ...node, path: repository.path, status: 'checking' }));
            this.nodes.push(...nodesWithPaths);

            nodesWithPaths.forEach(node => {
              setTimeout(() => {
              fetch(node.url, { method: 'HEAD' })
                .then(response => {
                  if (response.ok) {
                    node.status = 'ok';
                    console.log(node.name, node.status);
                    this.updateNodeStatus(node.name, 'ok');
                  } else {
                    node.status = 'error';
                    this.updateNodeStatus(node.name, 'error');
                  }
                })
                .catch(() => {
                  node.status = 'error';
                  this.updateNodeStatus(node.name, 'error');
                });
            }, 2000);

            });

          })
          .catch(error => console.error(error));
      });
    },
    updateNodeStatus(nodeName, status) {
  const nodeIndex = this.nodes.findIndex(node => node.name === nodeName);
  if (nodeIndex !== -1) {
    const updatedNode = { ...this.nodes[nodeIndex], status };
    this.nodes.splice(nodeIndex, 1, updatedNode);
  }
},


    filteredNodesByPath(path) {
      return this.nodes.filter(node => {
        const nodePath = node.path.toLowerCase();
        return (
          nodePath.includes(path.toLowerCase()) &&
          (this.pathFilter === '' || nodePath.includes(this.pathFilter.toLowerCase()))
        );
      });
    },
    setPathFilter(path) {
      this.pathFilter = path;
    },
  },
  computed: {
    filteredNodes() {
      if (this.pathFilter === '') {
        return this.nodes;
      }
      return this.filteredNodesByPath(this.pathFilter);
    },
    filteredPaths() {
      const paths = this.nodes.map(node => node.path);
      return [...new Set(paths)];
    },
  },
};
</script>