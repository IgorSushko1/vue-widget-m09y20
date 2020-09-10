<script>
    export default {
        data() {
            return {
                parentId: 'widget',
                parentElement: '',
                mainCurrency: 'RUB',
                currencyAmount: 5,
                currencyList: {},
                currencyKeys: [],
                marginX: 0,
                choosenCurrencies: ['USD', 'GBP', 'CHF', 'IDR'],
            };
        },
        async mounted() {
            this.getCurrencies('RUB');

            this.parentElement = document.getElementById(this.parentId);
        },
        methods: {
            async getCurrencies(name) {
                fetch(`https://api.exchangeratesapi.io/latest?base=${name}`)
                    .then((resp) => resp.json())
                    .then((data) => {
                        this.mainCurrency = name;
                        this.currencyList = data.rates;
                        if (this.mainCurrency !== 'EUR') {
                            this.currencyKeys = Object.keys(data.rates);
                        }
                    });
            },
            async Focused(name) {
                await this.getCurrencies(name);
                const lastElem = this.parentElement.querySelector(
                    '#' + this.mainCurrency
                );
                lastElem.classList.remove('currency-menu__name_focused');
                const elem = this.parentElement.querySelector('#' + name);
                elem.classList.toggle('currency-menu__name_focused');
                this.mainCurrency = name;
            },

            listMoveTo(where) {
                const elem = this.parentElement.querySelector(
                    '.currency-menu__list'
                );

                const liElems = this.parentElement.getElementsByTagName('li');
                const lengthOfOneLiElement = liElems[0].offsetWidth; //90
                const liLength = (liElems.length + 6) * lengthOfOneLiElement;

                const containerForList = this.parentElement.querySelector(
                    '.currency-menu__container-for-list'
                );
                const widthContainerForList = containerForList.getBoundingClientRect()
                    .width;

                if (where) {
                    if (where === 'prev') {
                        if (this.marginX > 0) {
                            this.marginX = this.marginX - 98;
                            elem.style.marginLeft = `-${this.marginX}px`;
                        }
                    }
                    if (where === 'next') {
                        const rightSum = this.marginX + widthContainerForList;

                        if (rightSum >= liLength) {
                            return;
                        } else {
                            this.marginX = this.marginX + 98;
                            elem.style.marginLeft = `-${this.marginX}px`;
                        }
                    }
                }
            },

            switchCurrency(where) {
                const removeFocus = () => {
                    const lastElem = this.parentElement.querySelector(
                        '#' + this.mainCurrency
                    );
                    lastElem.classList.remove('currency-menu__name_focused');
                };
                for (let i = 0; i < this.currencyKeys.length; i++) {

                    if (this.currencyKeys[i] === this.mainCurrency) {
                        if (where === 'prev' && i - 1 >= 0) {
                            removeFocus();
                            this.mainCurrency = this.currencyKeys[i - 1];
                            this.Focused(this.mainCurrency);

                            break;
                        } else if (
                            where === 'next' &&
                            i < this.currencyKeys.length - 1
                        ) {
                            removeFocus();
                            this.mainCurrency = this.currencyKeys[i + 1];
                            this.Focused(this.mainCurrency);

                            break;
                        }
                    }
                }
            },
        },
    };
</script>

<template>
    <div class="widget" id="widget">
        <article>
            <section>
                <div class="currency-menu">
                    <h3 class="currency-menu__header">
                        Курс {{ mainCurrency }} сегодня
                    </h3>
                    <div class="currency-menu__navigation">
                        <div
                            class="currency-menu__arrow_prev"
                            id="listMoveToPrev"
                            v-bind:class="{
                                'currency-menu__arrow_prev_active': marginX > 0,
                            }"
                            v-on:click="listMoveTo('prev', 'listMoveToPrev')"
                        >
                            &#8249;
                        </div>
                        <div class="currency-menu__container-for-list">
                            <ul class="currency-menu__list">
                                <li
                                    v-for="item in currencyKeys"
                                    :key="item"
                                    :id="item"
                                    v-on:click="Focused(item)"
                                    class="currency-menu__name"
                                >
                                    {{ item }}
                                </li>
                            </ul>
                        </div>
                        <div
                            class="currency-menu__arrow_next"
                            id="listMoveToNext"
                            v-on:click="listMoveTo('next', 'listMoveToNext')"
                        >
                            &#8250;
                        </div>
                    </div>
                </div>
            </section>
            <section class="widget__section-cards">
                <div class="input-container">
                    <input
                        type="text"
                        v-model="currencyAmount"
                        class="input-container__input"
                    />
                    <span class="input-container__currency">{{
                        mainCurrency
                    }}</span>
                </div>
                <div class="currency-recount__container">
                    <div
                        v-for="currency of choosenCurrencies"
                        v-bind:key="currency"
                        class="currency-recount__card"
                    >
                        <div class="currency-recount__title">
                            <h3>
                                <span class="currency-recount__currency-amount">
                                    {{ currencyAmount }}
                                </span>

                                <span class="currency-recount__currency-name">
                                    {{ mainCurrency }} =
                                </span>
                            </h3>
                        </div>
                        <div class="currency-recount__recounted">
                            <span class="currency-recount__recounted-numeral">
                                {{
                                    (
                                        currencyList[currency] * currencyAmount
                                    ).toLocaleString('ru-RU', {
                                        maximumFractionDigits: 2,
                                    })
                                }}
                            </span>
                            <span class="currency-recount__recounted-currency">
                                {{ currency }}
                            </span>
                        </div>
                    </div>
                </div>
                <div class="forward-back">
                    <div
                        class="forward-back__button forward-back__button_off"
                        v-on:click="switchCurrency('prev')"
                    >
                        <span class="forward-back__arrow">&#8249;</span> назад
                    </div>
                    <div
                        class="forward-back__button forward-back__button_on"
                        v-on:click="switchCurrency('next')"
                    >
                        далее <span class="forward-back__arrow">&#8250;</span>
                    </div>
                </div>
            </section>
        </article>
    </div>
</template>

<style lang="scss" scoped>
    @import url('https://fonts.googleapis.com/css2?family=Roboto+Condensed:wght@300;400;700&display=swap');

    .widget {
        min-width: 320px;
        max-width: 720px;
        font-family: 'Roboto Condensed', sans-serif;

        &__section-cards {
            margin-top: 32px;
        }
    }

    .currency-menu {
        height: 111px;
        background: #ffe782;
        min-width: 320px;
        max-width: 720px;
        overflow: hidden;

        &__header {
            padding: 0;
            margin: 0;
            text-align: left;
            padding-left: 24px;
            padding-top: 28px;
            font-size: 22px;
            letter-spacing: 0.5px;
            font-weight: 400;
        }

        &__navigation {
            display: flex;
            position: relative;
        }

        &__arrow_prev {
            position: absolute;
            font-size: 30px;
            transform-origin: 0 0;
            color: #ccae68;
            background: none;
            left: 26px;
            bottom: 8px;
            cursor: pointer;
            visibility: hidden;

            &_active {
                visibility: visible;
            }
        }

        &__arrow_next {
            position: absolute;
            font-size: 30px;
            transform-origin: 0 0;
            color: #ccae68;
            background: none;
            right: 17px;
            bottom: 8px;
            cursor: pointer;
        }

        &__container-for-list {
            margin: 0;
            margin-left: 40px;
            padding: 0;
            max-width: 635px;
            width: calc(100% - 60px);
            overflow: hidden;
        }

        &__list {
            display: flex;
            margin-left: 0px;
            margin: 0;
            padding: 0;
        }

        &__name {
            padding-top: 14px;
            min-width: 90px;
            height: 36px;
            list-style-type: none;
            text-align: center;
            margin-right: 8px;
            font-size: 14px;
            margin-top: 6px;
            color: #ccae68;
            font-weight: 400;
            letter-spacing: 1px;
            cursor: pointer;
        }
        &__name_focused {
            border-radius: 5px 5px 0 0;
            background: white;
        }
    }

    .input-container {
        width: 170px;
        float: right;
        position: relative;
        right: 18px;

        &__input {
            border: none;
            border-bottom: 1px solid #b9b9b9;
            text-align: right;
            font-size: 19px;
            width: 110px;
        }

        &__currency {
            color: #b9b9b9;
            margin-left: 9px;
            font-size: 18px;
        }
    }

    .currency-recount {
        &__container {
            display: flex;
            flex-wrap: wrap;
            justify-content: space-around;
            float: left;
            width: 100%;
            margin-top: 20px;
        }
        &__card {
            min-width: 274px;
            max-width: 327px;
            width: 100%;
            height: 138px;
            border-radius: 8px;
            box-shadow: 0 3px 3px 1px #efefef;
            overflow: hidden;
        }

        &__title {
            text-align: start;
            position: relative;
            left: 31px;
            top: -7px;
            font-size: 21px;
            letter-spacing: 1.1px;
        }

        &__currency-amount {
            color: #2b2d33;
            font-family: 'Roboto Condensed', sans-serif;
            font-size: 24px;
            font-weight: 300;
        }

        &__currency-name {
            color: #b9b9b9;
            font-weight: 100;
            font-family: 'Roboto Condensed', sans-serif;
            font-size: 24px;
        }

        &__recounted {
            text-align: start;
            position: relative;
            left: 35px;
            top: -15px;
            margin-top: 0px;
            letter-spacing: 2px;
        }

        &__recounted-numeral {
            font-size: 48px;
            font-weight: 400;
            font-family: 'Roboto Condensed', sans-serif;
        }

        &__recounted-currency {
            font-size: 24px;
            font-weight: 300;
            font-family: 'Roboto Condensed', sans-serif;
        }
    }

    .forward-back {
        display: flex;
        width: 100%;
        justify-content: center;
        position: relative;
        top: 57px;

        &__button {
            text-align: center;
            width: 112px;
            height: 32px;
            text-transform: uppercase;
            font-size: 15px;
            letter-spacing: 2px;
            padding: 2px;
            white-space: pre;
            border-radius: 8px;
            margin: 0 8px;
            box-shadow: 0 3px 3px 1px #efefef;
            margin-bottom: 30px;
            cursor: pointer;
        }
        &__button_off {
            background: #efefef;
            color: #787878;
        }
        &__button_on {
            background: #ffffff;
            color: #2b2d33;
        }
        &__arrow {
            font-size: 23px;
            position: relative;
            top: 1px;
            margin: 0 2px;
            cursor: pointer;
        }
        &__arrow_off {
            color: #787878;
        }
        &__arrow_on {
            color: #2b2d33;
        }
    }
</style>
