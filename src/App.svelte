<script lang="ts">
    import Peer from "peerjs";
    import { v7 } from "uuid";
    import { SHA256 } from "crypto-js";

    let formFields = $state({
        domain: "localhost:6969",
        username: v7(),
        password: "password",
    });

    let user = $derived(
        SHA256(formFields.username + ":" + formFields.password).toString(),
    );

    let messages: { timeStamp: string; message: string }[] = $state([]);

    let peer: Peer | undefined = $state();

    const addMessage = (message: string) => {
        const now = new Date();
        messages.push({
            timeStamp: `${now.getHours()}:${now.getMinutes()}:${now.getSeconds()}`,
            message,
        });
    };

    const submit = () => {
        const [host, port] = formFields.domain.split(":");

        peer = new Peer(user, {
            host,
            port: parseInt(port),
            path: "/peerjs",
        });

        peer.on("open", () => {
            addMessage(
                `Connected to peerserver on <i>${formFields.domain}/peerjs</i>`,
            );
        });

        peer.on("connection", (p) => {
            addMessage(`Connected to peer: ${p.peer}</i>`);
            p.send(user);
        });
    };
</script>

<div class="w-screen h-screen flex p-4">
    <fieldset class="fieldset p-4">
        <legend class="fieldset-legend">Domain</legend>
        <input
            type="text"
            class="input"
            defaultValue={formFields.domain}
            oninput={(e) =>
                (formFields = { ...formFields, domain: e.currentTarget.value })}
        />
        <legend class="fieldset-legend">Username</legend>
        <input type="text" class="input" defaultValue={formFields.username} />
        <legend class="fieldset-legend">Password</legend>
        <input
            type="password"
            class="input"
            defaultValue={formFields.password}
        />
        <button class="btn btn-primary" onclick={() => submit()}>Connect</button
        >
    </fieldset>
    <div class="flex-1 p-4">
        <div
            class="mockup-browser border border-base-300 w-full bg-base-200 h-full"
        >
            <div class="mockup-browser-toolbar">
                <div class="input">{formFields.domain}</div>
            </div>
            <div class="flex flex-col gap-2 p-8 overflow-y-scroll h-full">
                {#each messages as message}
                    <div class="block">
                        <span class="inline font-bold"
                            >{message.timeStamp}:</span
                        >
                        <span class="inline">{@html message.message}</span>
                    </div>
                {/each}
            </div>
        </div>
    </div>
</div>
