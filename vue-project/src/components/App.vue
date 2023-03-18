<template>
  <div id="app" class="dark-mode">
    <div class="nodes-container">
      <h2>Swiss Staking Load Balancer</h2>
      <div class="filter-container">
        <button v-for="chain in filteredChainTypes" :key="chain" @click="setchainTypeFilter(chain)">
          {{ chain }}
        </button>
      </div>

      <table class="nodes-table">
        <thead>
          {{ filteredNodes.length ? filteredNodes[0].chainType : '' }}.lb.swiss-staking.ch
          <i class="far fa-copy" @click="copyToClipboard"></i>

        </thead>
        <tbody>
          <div class="table-container">
          <tr v-for="node in filteredNodes" :key="node.name">
            <td>{{ node.address }}</td>
            <td>
              <span class="status-circle" :class="node.status"></span>
            </td>
          </tr>
        </div>
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
          username: 'cosmos',
          repository: 'chain-registry',
          chain: 'evmos',
          path: 'chain.json',
          type: 'rest'
        },
         {
          username: 'cosmos',
          repository: 'chain-registry',
          chain: 'evmos',
          path: 'chain.json',
          type: 'rpc'
         },
        // {
        //   username: 'creeea',
        //   repository: 'web3-load-balancer-configuration',
        //   path: 'evmos/lcd.json',
        // },
        // Add more repositories here...
      ],
      nodes: [],
      chainTypeFilter: '',
    };
  },
  created() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      this.repositories.forEach(repository => {
        const url = `https://api.github.com/repos/${repository.username}/${repository.repository}/contents/${repository.chain}/${repository.path}`;
        fetch(url)
          .then(response => response.json())
          .then(data => {
            const decodedData = JSON.parse(atob(data.content));
            const { apis: { [repository.type]: endpoints } } = decodedData;
            console.log(endpoints)
            const chainType = repository.chain+'-'+repository.type
            const endpointsWithStatus = endpoints.map(node => ({ ...node, chainType: chainType, status: 'checking' }));
            this.nodes.push(...endpointsWithStatus);

            endpointsWithStatus.forEach(node => {
              setTimeout(() => {
              fetch(node.url, { method: 'HEAD' })
                .then(response => {
                  if (response.ok) {
                    node.status = 'ok';
                    console.log("status ok", node.provider, node.address);
                    this.updateNodeStatus(node.provider, 'ok');
                  } else {
                    node.status = 'error';
                    this.updateNodeStatus("status down", node.provider, 'error');
                  }
                })
                .catch(() => {
                  node.status = 'error';
                  this.updateNodeStatus(node.provider, 'error');
                });
            }, 2000);

            });

          })
          .catch(error => console.error(error));
      });
    },
    updateNodeStatus(nodeName, status) {
  const nodeIndex = this.nodes.findIndex(node => node.provider === nodeName);
  if (nodeIndex !== -1) {
    const updatedNode = { ...this.nodes[nodeIndex], status };
    this.nodes.splice(nodeIndex, 1, updatedNode);
  }
},

    filteredNodesByChainType(chain) {
      return this.nodes.filter(node => {
        const nodeChain = node.chainType.toLowerCase();
        return (
          nodeChain.includes(chain.toLowerCase()) &&
          (this.chainTypeFilter === '' || nodeChain.includes(this.chainTypeFilter.toLowerCase()))
        );
      });
    },
    setchainTypeFilter(chain) {
      this.chainTypeFilter = chain;
    },

    copyToClipboard() {
  const heading = document.querySelector('.nodes-table thead').innerText;
  navigator.clipboard.writeText(heading);
},


  },

  computed: {
    filteredNodes() {
      if (this.chainTypeFilter === '') {
        return this.nodes;
      }
      return this.filteredNodesByChainType(this.chainTypeFilter);
    },
    filteredChainTypes() {
      const chains = this.nodes.map(node => node.chainType);
      return [...new Set(chains)];
    },
  },
};
</script>
<style src="../assets/AppStyles.css"></style>

