<template>
    <div>
        <div class="jumbotron">
            <b-container class="">
                <b-row>
                    <b-col md="3">
                        <b-form>
                            <label v-if="!isInCommitment()">Commitment Process is not started yet.</label>
                            <div id="commitmentDiv" v-if="isInCommitment()">
                                <h4>Want to try your luck?</h4>
                                <p>Quick! Less than <b>{{(numOfBlockLeftForCommitment() + 1) * 15}} seconds </b> left
                                    to buy a ticket.</p>
                                <label>Enter the number</label>
                                <b-input type="number" id="revealNumber" aria-describedby="revealNumber"
                                         v-model="revealNumber"
                                         placeholder="Enter your number"></b-input>
                                <span class="text-muted"><small><i>Ticket price = <b>0.01 ETH</b></i></small></span>
                                <div class="d-flex justify-content-end flex-column">
                                    <b-button variant="primary" size="sm" class="mt-2 mb-2 text-white"
                                              @click.prevent="commitTicket()">Join the lottery
                                        <b-spinner label="Loading..." small v-show="loading==true"></b-spinner>
                                    </b-button>

                                </div>
                            </div>

                            <label v-if="!isInReveal() && isInCommitment()">Reveal Process is not started yet. It will
                                start in
                                {{numOfBlockLeftForCommitment()*15}} seconds.</label>
                            <div id="revealDiv" v-if="isInReveal()">
                                <label v-if="isInReveal()">Reveal your number</label>
                                <p>Quick only <b>{{(numOfBlockLeftForReveal() + 1) * 15}} seconds</b> left to reveal
                                    your
                                    number</p>
                                <b-input type="number" id="revealNumber" aria-describedby="revealNumber"
                                         v-model="revealNumber"
                                         placeholder="Enter your number"></b-input>
                                <div class="d-flex justify-content-end flex-column">
                                    <b-button variant="primary" size="sm" class="mt-2 mb-2 text-white"
                                              @click.prevent="reveal()">Reveal
                                        <b-spinner label="Loading..." small v-show="loading==true"></b-spinner>
                                    </b-button>

                                </div>
                            </div>
                        </b-form>
                        <hr>

                        <b-form class="mt-4" v-if="isInPickAWinner()">
                            <b-button id="getWinnerButton" block variant="success" @click.prevent="getWinner()"
                                      :disabled="userAccount!==manager && isInPickAWinner()">Pick a Winner
                                <b-spinner label="Loading..." small v-show="loading2==true"></b-spinner>
                            </b-button>
                        </b-form>

                        <div id="withdrawPrize" v-if="isCurrentUserWinner()">
                            <b-form class="mt-4">
                                <label>CONGRATS!!! You are the WINNER! You won <b>ETH: {{totalPrize}} </b></label>
                                <b-button block variant="success" @click.prevent="withdrawPrize()"
                                          :disabled="!isCurrentUserWinner()">Claim your prize
                                    <b-spinner label="Loading..." small v-show="loading3==true"></b-spinner>
                                </b-button>
                            </b-form>
                        </div>
                    </b-col>
                    <b-col cols="6" md="5">
                        <h4>Lottery info</h4>
                        <p>Total Amount in the pool: ETH
                            <b-badge variant="success">{{balance - totalPrize}}</b-badge>
                        </p>
                        <b-alert variant="primary" show><small>
                            <b-icon icon="star-fill" variant="warning"></b-icon>
                            Previous Winner <br>
                            <svg class="svg-icon" viewBox="0 0 20 20" width="24" height="24">
                                <path d="M12.075,10.812c1.358-0.853,2.242-2.507,2.242-4.037c0-2.181-1.795-4.618-4.198-4.618S5.921,4.594,5.921,6.775c0,1.53,0.884,3.185,2.242,4.037c-3.222,0.865-5.6,3.807-5.6,7.298c0,0.23,0.189,0.42,0.42,0.42h14.273c0.23,0,0.42-0.189,0.42-0.42C17.676,14.619,15.297,11.677,12.075,10.812 M6.761,6.775c0-2.162,1.773-3.778,3.358-3.778s3.359,1.616,3.359,3.778c0,2.162-1.774,3.778-3.359,3.778S6.761,8.937,6.761,6.775 M3.415,17.69c0.218-3.51,3.142-6.297,6.704-6.297c3.562,0,6.486,2.787,6.705,6.297H3.415z"></path>
                            </svg>
                            {{ lastWinner }}</small>
                            <h3>Won:{{ totalPrize }} ETH </h3>
                        </b-alert>
                        <div id="revealTicketsDiv" v-if="isInReveal()">
                            <h6>Revealed Tickets</h6>
                            <p>There are currently {{numOfRevealedTickets}} tickets revealed. Addresses of tickets: </p>
                            <b-list-group>
                                <b-list-group-item class="d-flex justify-content-between align-items-center"
                                                   v-for="index in revealedTicketsAddresses" :key="index">
                                    <svg class="svg-icon" viewBox="0 0 20 20" width="24" height="24">
                                        <path d="M12.075,10.812c1.358-0.853,2.242-2.507,2.242-4.037c0-2.181-1.795-4.618-4.198-4.618S5.921,4.594,5.921,6.775c0,1.53,0.884,3.185,2.242,4.037c-3.222,0.865-5.6,3.807-5.6,7.298c0,0.23,0.189,0.42,0.42,0.42h14.273c0.23,0,0.42-0.189,0.42-0.42C17.676,14.619,15.297,11.677,12.075,10.812 M6.761,6.775c0-2.162,1.773-3.778,3.358-3.778s3.359,1.616,3.359,3.778c0,2.162-1.774,3.778-3.359,3.778S6.761,8.937,6.761,6.775 M3.415,17.69c0.218-3.51,3.142-6.297,6.704-6.297c3.562,0,6.486,2.787,6.705,6.297H3.415z"></path>
                                    </svg>
                                    <small>{{ index }}</small>
                                </b-list-group-item>
                            </b-list-group>

                        </div>
                    </b-col>
                    <b-col cols="6" md="4">Contract Details
                        <b-card>
                            <p>Your Current balance is <br>ETH:
                                <b-badge variant="success">{{userBalance}}</b-badge>
                            </p>
                            <p>Your Account Address is:
                                <b-alert variant="success" show>{{ userAccount }}
                                </b-alert>
                            </p>
                            <hr>
                            <p>This Contract is managed by
                                <b-alert variant="primary" show><small> {{ manager }}</small></b-alert>
                            </p>
                        </b-card>
                    </b-col>
                </b-row>
            </b-container>
        </div>
    </div>
</template>
<script>
    import lottery from '../contract'
    import web3 from '../web3'

    export default {
        blockNumber: Number,
        revealDeadline: Number,
        ticketDeadline: Number,
        data() {
            return {
                totalPrize: '',
                manager: '',
                userBalance: '',
                userAccount: '',
                balance: '',
                numOfRevealedTickets: Number,
                revealedTicketsAddresses: [],
                revealNumber: '',
                lastWinner: '',
                error: false,
                loading: false,
                loading2: false,
                loading3: false
            }
        },
        async created() {
            await web3.eth.getBlockNumber((error, result) => {
                if (error) {
                    console.log(error)
                } else {
                    this.blockNumber = result
                }
            });
            lottery.methods.ticketDeadline().call().then(val => {
                this.ticketDeadline = val;
                console.log(this.ticketDeadline)
            });
            lottery.methods.revealDeadline().call().then(val => {
                this.revealDeadline = val;
                console.log(this.revealDeadline)
            });
            lottery.methods.totalPrize().call().then(val => {
                this.totalPrize = web3.utils.fromWei(val, 'ether');
                console.log(this.revealDeadline)
            });
            this.getLastWinner();
            //Update contract manager
            this.manager = await lottery.methods.manager().call();
            //Revealed Tickets
            this.getNumOfRevealedTickets();

            //Contract Balance
            this.getBalanceOfContract();

            //User Account
            const accounts = await web3.eth.getAccounts();
            this.userAccount = accounts[0];
            console.log('user account:' + this.userAccount);

            //User Balance
            const cbal = await web3.eth.getBalance(this.userAccount);
            this.userBalance = web3.utils.fromWei(cbal, 'ether');
            console.log(this.userBalance)
        },
        watch: {
            userAccount: function () {
                window.ethereum.on('accountsChanged', () => {

                    web3.eth.getAccounts().then(acc => {
                        this.userAccount = acc[0]
                        console.log('[ACCOUNT CHANGED]', this.userAccount)

                        web3.eth.getBalance(this.userAccount).then(bal => {
                            this.userBalance = web3.utils.fromWei(bal, 'ether')
                            console.log('[Balance CHANGED]', this.userBalance)
                        })
                        this.$toast('Account Changed', {
                            timeout: 1000,
                            type: 'success'
                        });
                    })
                })
            },
        },
        methods: {
            async commitTicket() {
                this.loading = true;

                console.log('userAccount' + this.userAccount + 'revealNumber' + this.revealNumber);
                await lottery.methods.createCommitment(this.revealNumber).send({
                    from: this.userAccount,
                    value: web3.utils.toWei('0.01', 'ether')
                });
                await this.getBalanceOfContract();
                await this.getNumOfRevealedTickets();
                this.$toast('Successfully committed the ticket.', {
                    timeout: 1000,
                    type: 'success'
                });

                this.loading = false
            },
            async reveal() {
                this.loading = true;

                console.log('userAccount' + this.userAccount + 'revealNumber' + this.revealNumber);
                await lottery.methods.reveal(this.revealNumber).send({
                    from: this.userAccount,
                });
                await this.getBalanceOfContract();
                await this.getNumOfRevealedTickets();
                this.$toast('Successfully revealed the ticket.', {
                    timeout: 1000,
                    type: 'success'
                });

                this.loading = false
            },
            getBalanceOfContract() {
                web3.eth.getBalance(lottery.options.address).then(bal => {
                    return this.balance = web3.utils.fromWei(bal, 'ether')
                })
            },
            async getNumOfRevealedTickets() {
                await lottery.methods.getNumberOfRevealedTickets().call().then(p => {
                    this.numOfRevealedTickets = p
                    console.log(this.numOfRevealedTickets)
                });
                this.revealedTicketsAddresses = [];
                for (let i = 0; i < this.numOfRevealedTickets; i++) {
                    await lottery.methods.getRevealedTicket(i).call().then(p => {
                        this.revealedTicketsAddresses.push(p);
                    });
                }
                return this.numOfRevealedTickets
            },
            getLastWinner() {
                lottery.methods.winner().call().then(val => {
                    this.lastWinner = val;
                    console.log(this.lastWinner)
                })
            },
            isInReveal() {
                return this.blockNumber > this.ticketDeadline && this.blockNumber <= this.revealDeadline;
            },
            isInCommitment() {
                return this.blockNumber <= this.ticketDeadline;
            },
            isInPickAWinner() {
                return this.blockNumber > this.revealDeadline;
            },
            isCurrentUserWinner() {
                return this.lastWinner === this.userAccount;
            },
            numOfBlockLeftForCommitment() {
                return this.ticketDeadline - this.blockNumber;
            },
            numOfBlockLeftForReveal() {
                return this.revealDeadline - this.blockNumber;
            },
            async getWinner() {
                this.loading2 = true;
                await lottery.methods.drawWinner().send({
                    from: this.userAccount
                })
                this.$toast('Winner Winner Chicken Dinner!', {
                    timeout: 2000,
                    type: 'success'
                });
                this.loading2 = false;
                this.getLastWinner();
                await this.getBalanceOfContract()
            },
            async withdrawPrize() {
                this.loading3 = true;
                await lottery.methods.withdrawPrize().send({
                    from: this.userAccount
                })
                this.$toast('Your prize successfully sent to your account!', {
                    timeout: 2000,
                    type: 'success'
                });
                this.loading3 = false;
                this.getLastWinner();
                await this.getBalanceOfContract()
            }
        }
    }
</script>
