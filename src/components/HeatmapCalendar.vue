<template>
    <div class="wrapper_calendar">
        <h1 id="heading1">Change Heat Map</h1>
        <div id="heatmap_clendar">
        </div>
        <div class="color_range">
            <span>Less</span>
            <div v-for="heatColor in heatColors" :key="heatColor" class="color_of_range"
                :style="{ 'background-color': heatColor }">
            </div>
            <span>More</span>
        </div>
    </div>
</template>   
<script lang="ts">

import countObject from "../types/countObject"
import { defineComponent, onBeforeMount, onMounted, reactive, ref } from 'vue';
import count from "../types/countObject";

export default defineComponent({
    emits: ["ChangeOnDate"],
    setup(props, context) {
        const startDate = ref("")
        const endDate = ref("")
        var month = ref(0)
        var year = ref(0)
        const heatColors = ref<string[]>(["#E6E6E6", "#fdb788", "#ffaa71", "#f8690a", "#FC6600"])
        const months = ref(["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"]);

        let DataOfCounts = ref<count>({})
        let getDate = reactive({}) as Date
        let terminateDate = reactive({}) as Date

        onMounted(() => {
            getDatesOfChanges();


        })
        function callToCreateCalendar() {

            
            while (year.value < terminateDate.getFullYear() || month.value < terminateDate.getMonth()+1) {
                console.log(month.value, "laast")
                createCalendar(document.getElementById('heatmap_clendar'), year.value, month.value);
                if (month.value < 11) {
                    month.value += 1
                } else if (month.value == 11) {
                    year.value += 1
                    month.value = 0
                }
            }

            let allDateElem = document.querySelectorAll<HTMLElement>('.single_div')
            allDateElem.forEach((elem: HTMLElement) => {
                elem.addEventListener("click", () => {

                    let getPopup = elem.firstElementChild as HTMLElement;

                    if (getPopup) {
                        let innerChild = getPopup.firstElementChild as HTMLElement
                        let innerContent = innerChild.innerHTML
                        context.emit("ChangeOnDate", innerContent)
                    }

                })
            })
        }

        //append the element into the table
        function createCalendar(elem: HTMLElement | null, year: number, month: number) {


            let d = new Date(year, month);
            getDate = d;
            let table = '<p>' + months.value[d.getMonth()] + '</p><table><tr>';

            // spaces for the first row
            // from Monday till the first day of the month
            for (let i = 0; i < getDay(d); i++) {
                table += '<td></td>';
            }
            // <td> with actual dates
            while (d.getMonth() == month) {


                let getChanges = `${d.toISOString().split('T')[0]}`
                let countOfChanges = 0;
                let color = ''


                if (Object.keys(DataOfCounts.value).includes(getChanges)) {

                    countOfChanges = DataOfCounts.value[getChanges]

                } else {
                    countOfChanges = 0
                }
                if (countOfChanges < 1) {
                    color = heatColors.value[0]
                }
                if (countOfChanges >= 1 && countOfChanges <= 5) {
                    color = heatColors.value[1]
                }
                if (countOfChanges >= 6 && countOfChanges < 8) {
                    color = heatColors.value[2]
                }
                if (countOfChanges >= 8 && countOfChanges < 15) {
                    color = heatColors.value[3]
                }
                if (countOfChanges >= 15) {
                    color = heatColors.value[4]
                }

                table += `<td style="background-color:${color}" class="single_div">
                            <div class="popup">
                                <p class="single_div_date">${d}</p>
                                <p class="inner">${countOfChanges} changes on ${d.toLocaleString().split(',')[0]}</p>
                            </div>
                        </td>`;

                if (getDay(d) % 7 == 6) { // sunday, last day of week - newline
                    table += '</tr><tr>';
                }

                d.setDate(d.getDate() + 1);
            }



            // add spaces after last days of month for the last row
            if (getDay(d) != 0) {
                for (let i = getDay(d); i < 7; i++) {
                    table += '<td></td>';
                }
            }

            // close the table
            table += '</tr></table>';
            const node = document.createElement("div");
            node.innerHTML = table;

            elem ? elem.appendChild(node) : null;
        }

        function getDay(date: any) { // get day number from 0 (monday) to 6 (sunday)
            let day = date.getDay();
            if (day == 0) day = 7; // make Sunday (0) the last day
            return day - 1;
        }

        //fetching the data from API
        const getDatesOfChanges = () => {
            fetch('https://heatmapchanges.free.beeceptor.com/changes/all').then((response) => {
                if (response.ok) {
                    return response.json();
                }
                throw new Error('Something went wrong');
            })
                .then((result) => {
                    startDate.value = result.startDate
                    endDate.value = result.endDate
                    DataOfCounts.value = result.counts
                    getDate = new Date(startDate.value)
                    terminateDate = new Date(endDate.value)
                    month.value = getDate.getMonth()
                    year.value = getDate.getFullYear()
                    callToCreateCalendar();

                })
                .catch((error) => {
                    console.log(error)
                });
        }

        return {
            heatColors
        }
    },

});
</script>
<style>
.wrapper_calendar {
    position: relative;
    margin: 32px 60px;
    padding-top: 20px;
    padding-right: 50px;
    padding-left: 50px;
    padding-bottom: 70px;
    border: 1px solid #fff;
    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}

#heading1 {
    color: rgb(54, 8, 97);
    margin: 0;
    margin-bottom: 30px;
}

.color_range {
    position: absolute;
    right: 100px;
    display: flex;
    flex-direction: row;
    bottom: 23px;
}

.color_of_range {
    height: 10px;
    width: 10px;
}

td,
th {
    padding: 3px;
    height: 10px;
    width: 10px;
    text-align: center;
}

th {
    font-weight: bold;
}

#heatmap_clendar {
    display: flex;
}

.single_div {
    margin: 150px auto 20px;
    position: relative;
}

.popup {
    display: none;
    position: absolute;
    left: -80px;
    top: -29px;
    z-index: 1;
    padding: 0px 3px;
    border-radius: 5px;
    font-size: 13px;
    width: 162px;
    transform: translate3d(0, -50%, 0);
    box-shadow: 2px 2px 8px rgb(0 0 0 / 50%);
    background: rgb(0, 0, 0);
    color: #fff;
}

.popup:after {
    content: "";
    width: 20px;
    height: 20px;
    transform: rotate(-45deg);
    background: rgba(0, 0, 0);
    ;
    position: absolute;
    box-shadow: 1px 4px 8px rgba(0, 0, 0, 0.5);
    z-index: -1;
    bottom: -10px;
    left: calc(50% - 10px);
}

.inner {
    border-radius: 100px;
}

.single_div:hover .popup {
    display: block;
}

.single_div_date {
    display: none;
}
</style>