# hakka-forum-themes

### edit path

https://your.discourse.url/admin/customize/themes/1/common/scss/edit


### css

```
button[title="Revoke"] {
  display: none;
}

button.sign-up-button {
  display: none;  
}

div.voting-power > span > a {
  color: #aa7c0a;
}
```

### head

```
<script src="https://cdn.ethers.io/lib/ethers-5.1.umd.min.js" type="text/javascript">
</script>

<script type="text/discourse-plugin" version="0.8">
    const tokenAddress = '0xd9958826bce875a75cc1789d5929459e6ff15040';
    const alchemyKey = 'st8j5vqpEXjaM2gcdcR6uQZoyLN1gwXS'; // from https://www.alchemy.com/
    const provider = new ethers.providers.AlchemyProvider('homestead', alchemyKey);
    const abi = ["function votingPower(address address) view returns (uint256)"];
    const token = new ethers.Contract(tokenAddress, abi, provider);
    
    const h = require("virtual-dom").h;

    function numberWithCommas(x) {
        return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    }

    api.createWidget("voting-power", {
        buildKey: () => 'voting-power',
        defaultState() {
            return { power: null };
        },
        
        updateVotingPower() {
            this.state.power = -1; // KEY POINT TO AVOID LOOP WHEN mount-widget
            return new Promise(async (resolve, reject) => {
                try {
                    const address = await provider.resolveName(this.attrs.username);
                    const power = await token.votingPower(address); // use token.balanceOf(address) if it's a simple ERC20 token
                    this.state.power = numberWithCommas(Math.floor(power.toString() / 1e18));
                }
                catch (e) {
                    this.state.power = 0;
                }
                resolve();
            });
        },
        

        html(attrs, state) {
            if (this.state.power === null) {
                this.sendWidgetAction("updateVotingPower");
            }
            else {
                return h("div.voting-power".concat(this.attrs.isUserCard ? '' : '.topic-meta-data.trigger-user-card'),
                    h("span.second.full-name",
                        h("a", 
                            {
                                innerText: state.power === -1 
                                    ?   `VP: Loading...` 
                                    :   `VP: ${this.state.power}`,
                                attributes: {
                                    href: attrs.usernameUrl,
                                    "data-user-card": attrs.username,
                                }
                            }
                        )
                    )
                );
            }
        }
    })

    api.decorateWidget('post-body:after-meta-data', (helper) => {
        const username = helper.getModel().get('username');
        const usernameUrl = helper.getModel().get('usernameUrl');
        return helper.widget.attach('voting-power', {
            isUserCard: false,
            username,
            usernameUrl
        });
    });
</script>

<script type="text/x-handlebars" data-template-name="/connectors/user-card-post-names/voting-power">
    {{mount-widget widget="voting-power" args=(hash username=user.username usernameUrl=user.usernameUrl isUserCard=true)}}
</script>
<script type="text/x-handlebars" data-template-name="/connectors/user-post-names/voting-power">
    {{mount-widget widget="voting-power" args=(hash username=model.username usernameUrl=model.usernameUrl isUserCard=true)}}
</script>


```
