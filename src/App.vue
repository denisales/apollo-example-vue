<template>
    <div id="app">
        <div id="nav">
            <router-link to="/">Home</router-link> |
            <router-link to="/about">About</router-link>
        </div>
        <router-view/>
        <table>
          <tr>
            <td>id</td>
            <td>nome</td>
            <td>nome amigo</td>
          </tr>
          <tr v-for="dado in dados" :key="dado.id">
            <td>{{dado.id}}</td>
            <td>{{dado.first_name}}</td>
            <td>{{dado.friends.name}}</td>
          </tr>
        </table>
    </div>
</template>


<script>
import { ApolloClient } from "apollo-client";
import { InMemoryCache } from "apollo-cache-inmemory";
import { RestLink } from "apollo-link-rest";
import gql from "graphql-tag";

export default {
  data() {
    return {
      dados: []
    }
  },
    mounted() {
        // setup your `RestLink` with your endpoint
        // const restLink = new RestLink({ uri: "https://reqres.in/api" });
        const restLink = new RestLink({ endpoints: { v1: 'https://reqres.in/api', v2: 'https://www.swapi.co/api' } });

        // setup your client
        const client = new ApolloClient({
            link: restLink,
            cache: new InMemoryCache()
        });

        const query = gql `
      query GET_USERS {
        users @rest(type: "Users", path: "/users", method: "GET", endpoint: "v1") {
          total
          data @type(name: "User") {
            id @export(as: "id")
            first_name
             friends @rest(path: "/people/{exportVariables.id}", type: "[User]", endpoint: "v2") {
              name
            }
          }
        }
      }
    `;

        client.query({ query }).then(response => {
          let dados = response.data.users.data
          dados.forEach(i => {
            this.dados.push(i) 
          });
        });
    }
};
</script>

<style lang="scss">
#app {
    font-family: "Avenir", Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
}

#nav {
    padding: 30px;
    a {
        font-weight: bold;
        color: #2c3e50;
        &.router-link-exact-active {
            color: #42b983;
        }
    }
}
</style>
