<script lang="ts">
    import { push } from "svelte-spa-router";
    import LayoutGrid, { Cell } from "@smui/layout-grid";
    import Paper, { Title, Subtitle, Content } from "@smui/paper";
    import IconButton, { Icon } from "@smui/icon-button";
    import { axiosInstance, sourceURL } from "../functions/source";
    import { beforeUpdate, onMount } from "svelte";

    const config = {
        method: "get",
        url: `${sourceURL}/classroom/`,
        headers: {},
    };

    let classList = [];
    onMount(async () => {
        await axiosInstance(config)
            .then((response) => {
                classList = response.data;
                console.log(response.data);
            })
            .catch((error) => {
                console.log(error);
            });
    });
</script>

<LayoutGrid>
    {#each classList as classroom, i}
        <Cell span={2}>
            <Paper>
                <div>
                    <Title>{classroom.name}</Title>
                    <IconButton
                        class="material-icons"
                        on:click={() => push("/class/student")}
                    >
                        groups
                    </IconButton>
                    <IconButton class="material-icons">delete</IconButton>
                </div>
                <Subtitle>교수자 : {classroom.professor_name}</Subtitle>
                <Subtitle>년도 : {classroom.year}</Subtitle>
                <Subtitle>학기 : {classroom.semester}</Subtitle>
            </Paper>
        </Cell>
    {/each}
    <Cell span={1}>
        <Paper class="add-paper">
            <IconButton
                class="material-icons"
                on:click={() => push("/class/new")}>add</IconButton
            >
        </Paper>
    </Cell>
</LayoutGrid>

<style>
    :global(.add-paper) {
        display: flex;
        align-items: center;
        flex-direction: row;
        justify-content: center;
        height: 100%;
    }
</style>
