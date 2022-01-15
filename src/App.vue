<template>
    <div id="app">
        <div id="nav" class="text-center">
            <router-link to="/">Home</router-link>
            |
            <router-link to="/lottery">Lottery Game</router-link>
        </div>
        <router-view/>
    </div>
</template>

<style>

    #nav a.router-link-exact-active {
        color: #42b983;
    }


    #nav {
        padding: 30px;
    }

    #nav a {
        font-weight: bold;
        color: #2c3e50;
    }

    #app {
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        color: #2c3e50;
    }

</style>
