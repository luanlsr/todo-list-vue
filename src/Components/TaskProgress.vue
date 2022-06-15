<template>
    <div class="tasks-progress">
        <span class="progress-value">{{progress}}%</span>
        <div class="progress-bar" :style="{width: progress + '%'}"></div>
    </div>
</template>

<script>
    export default {
        props: {
            progress: {typee: Number, default: 0}
        },
    }
</script>

<style>
    .tasks-progress {
        position: relative;
        width: 30%;
        color: #fff;
        border: 1px solid #fff;
        border-radius: 8px;
        margin-bottom: 15px;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .progress-bar {
        position: absolute;
        height: 100%;
        border-radius: 8px;
        background-color: #4caf50;
        align-self: flex-start;
    }

    .progress-value{
        z-index: 1;
        font-size: 1.7rem;
    }
</style>
