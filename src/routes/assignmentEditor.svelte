<script>
    import { axiosInstance } from "../functions/source.js";
    import { onMount } from "svelte";
    import Textfield from "@smui/textfield";
    import HelperText from "@smui/textfield/helper-text";
    import IconButton from "@smui/icon-button";
    import Button from "@smui/button";
    import { DateInput } from "date-picker-svelte";
    import Select, { Option } from "@smui/select";
    import Card, { Content } from "@smui/card";
    import Testcase from "../component/Testcase.svelte";
    import Paper, { Title, Subtitle } from "@smui/paper";
    import { push } from "svelte-spa-router";

    import Quill from "quill";
    let quill = null;

    onMount(async() => {
        let container = document.getElementById("editor");
        quill = new Quill(container, {
            modules: {
                toolbar: [
                    [
                        { header: 1 },
                        { header: 2 },
                        "blockquote",
                        "link",
                        "image",
                        "video",
                    ],
                    ["bold", "italic", "underline", "strike"],
                    [{ list: "ordered" }, { list: "ordered" }],
                    [{ align: [] }],
                    ["clean"],
                ],
            },
            placeholder: "Type something...",
            theme: "snow", // or 'bubble'
        });

        try {
            console.log("request prof classroom request");
            await axiosInstance.get("/classroom/").then(res => {
                for (let i = 0; i < res.data.length; i++) {
                    classes[i] = {
                        id: res.data[i].id, 
                        className: res.data[i].name
                    };
                }
            }).catch(err => {
                console.log("request prof classroom request fail : " + err);
            }).finally(() =>{
                console.log("request prof classroom request end");
            });
        } catch(err) {
            console.log(err);
        }
    });

    let title = "";
    let classInfo = "";
    let start = new Date();
    let end = new Date();

    // let classes = ["자연어처리", "알고리즘", "프로그래밍 입문"];
    let classes = [];

    let testcases = [];
    let testcaseInput = "";
    let testcaseOutput = "";
    let testcaseScore ="";

    function handleKeydown(event) {
        console.log(event.key);

        if (event.keyCode == 13) {
            console.log(testcases);
            addTestCase();
            console.log(testcases);
        }
    }

    function addTestCase() {
        testcases = [
            ...testcases,
            { input: testcaseInput, output: testcaseOutput, score: testcaseScore},
        ];
        testcaseInput = "";
        testcaseOutput = "";
        testcaseScore = "";

    }

    function deleteCase(testcase) {
        testcases = testcases.filter((element) => element !== testcase);
    }

    function leftPad(value) {
        if (value >= 10) {
            return value;
        }

        return `0${value}`;
    }

    function convertDateFormat(source) {
        source.setHours(source.getHours() + 9);
        return source.toISOString().substring(0, 23);
    }

    async function addAssignment() {
        try {
            const data = {
                "classroom_id": classInfo.id,
                "title": title,
                "desc": quill.getText(0, quill.getLength()-1),
                "start_date": convertDateFormat(start), //UTC
                "end_date": convertDateFormat(end), //UTC
                "parent_id": null
            };
            
            console.log("request prof add assignment request");
            await axiosInstance.post("/assignment/", data).then(res => {
                addAssignmentTestCase(res.data.id);
                push("/class/" + classInfo.id + "/assigment");
            }).catch(err => {
                console.log("request prof add assignment request fail : " + err);
            }).finally(() =>{
                console.log("request prof add assignment request end");
            });
        } catch(err) {
            console.log(err);
        }
    }

    async function addAssignmentTestCase(assignment_id) {
        try {
            let data = {
                "assignment_id": assignment_id,
            };
            
            console.log("request prof add assignment testcase request");
            for (let i = 0; i < testcases.length; i++) {
                data = {
                    ...data,
                    "input": testcases[i].input,
                    "output": testcases[i].output,
                }
                console.log(data)
                await axiosInstance.post("/testcase/", data)
                    .catch(err => {
                        console.log("request prof add assignment testcase request fail : " + err);
                    }).finally(() =>{
                        console.log("request prof add assignment testcase request end");
                    });
            }
        } catch(err) {
            console.log(err);
        }
    }
</script>

<svelte:head>
    <link href="//cdn.quilljs.com/1.3.6/quill.snow.css" rel="stylesheet" />
</svelte:head>

<!-- <svelte:window on:keydown={handleKeydown}/> -->
<Paper class="assignment-editor-paper">
    <div class="assignment-editor-subject-title"> 
        <Select bind:value={classInfo} label="과목명" class="assignment-editor-subject">
            {#each classes as class_name}
                <Option value={class_name}>{class_name.className}</Option>
            {/each}
        </Select>

        <Textfield bind:value={title} label="제목" class="assignment-editor-title">
            <HelperText slot="helper">Helper Text</HelperText>
        </Textfield>
    </div>

    <div class="assignment-editor-start-date">
        <h1>시작일</h1>
        <DateInput bind:value={start} format="yyyy-MM-dd HH:mm" />
    </div>
    
    <div class="assignment-editor-end-date">
        <h1>종료일</h1>
        <DateInput bind:value={end} format="yyyy-MM-dd HH:mm" />
    </div>

    <main>
        <div id="editor" />
    </main>

    <div class="assignment-editor-testcase">
        <Title>테스트 케이스</Title>

        {#each testcases as testcase}
            <Testcase input={testcase.input} output={testcase.output} score={testcase.score}>
                <IconButton class="material-icons" on:click={deleteCase(testcase)}
                    >delete</IconButton
                >
            </Testcase>
            <br />
        {/each}

        <div class="assignment-editor-testcase-input">
            <Textfield
                textarea
                bind:value={testcaseInput}
                label="입력"
                input$rows={4}
                input$cols={24}
                input$resizable={false}
            >
                <HelperText slot="helper">테스트케이스의 입력을 넣어주세요.</HelperText>
            </Textfield>
        </div>

        <div class="assignment-editor-testcase-output">
            <Textfield
                textarea
                bind:value={testcaseOutput}
                label="출력"
                input$rows={4}
                input$cols={24}
                input$resizable={false}
            >
                <HelperText slot="helper"
                    >테스트케이스의 출력을 넣어주세요.</HelperText
                >
            </Textfield>
        </div>

        <!-- <div class="assignment-editor-testcase-score">
            <Textfield
                textarea
                bind:value={testcaseScore}
                label="점수"
                input$rows={4}
                input$cols={7}
                input$resizable={false}
                on:keydown={handleKeydown}
            >
                <HelperText slot="helper"
                    >테스트케이스의 점수를 넣어주세요.</HelperText
                >
            </Textfield>
        </div> -->

        <IconButton class="material-icons assignment-editor-testcase-add" on:click={addTestCase}
            >add</IconButton
        >
    </div>
</Paper>
<br />

<Button variant="raised" class="button-shaped-round" on:click={addAssignment}>제출</Button>

<style>
    main {
        text-align: center;
        padding: 1em;
        max-width: 240px;
        margin: 0 auto;
    }
  
    #editor {
        height: 350px;
    }

    @media (min-width: 640px) {
        main {
            max-width: none;
        }
    }
</style>
