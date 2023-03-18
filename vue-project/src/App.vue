<template>
  <div id="app" class="dark-mode">
    <div class="nodes-container">
      <h2>Available Nodes</h2>
      <div class="filter-container">
        <input
          type="text"
          v-model="searchQuery"
          placeholder="Filter nodes..."
        />
      </div>
      <ul v-for="path in filteredPaths" :key="path">
        <h3>{{ path }}</h3>
        <li>
          <table>
            <thead>
              <tr>
                <th>Name</th>
                <th>Website</th>
                <th>URL</th>
                <th>Keybase</th>
                <th>Weight</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="node in filteredNodesByPath(path)" :key="node.name">
                <td>{{ node.name }}</td>
                <td>{{ node.url }}</td>
                <td>{{ node.keybase }}</td>
                <td>{{ node.weight }}</td>
              </tr>
            </tbody>
          </table>
        </li>
      </ul>
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
      searchQuery: '',
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
            const nodesWithPaths = decodedData.map(node => ({ ...node, path: repository.path }));
            this.nodes.push(...nodesWithPaths); // Append the nodes with path to the nodes array
          })
          .catch(error => console.error(error));
      });
    },
    filteredNodesByPath(path) {
      return this.nodes.filter(node => {
        const searchQuery = this.searchQuery.toLowerCase();
        const nodePath = node.path.toLowerCase();
        return (
          nodePath.includes(path.toLowerCase()) &&
          nodePath.includes(searchQuery)
        );
      });
    },
  },
  computed: {
    filteredPaths() {
      const paths = this.nodes.map(node => node.path);
      return [...new Set(paths)];
    },
  },
};
</script>

<style src="./AppStyles.css"></style>
