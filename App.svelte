<!-- App.svelte -->
<script>
  import { onMount, onDestroy } from "svelte";

  const MARKET = "SUSHI-PERP";

  let socket = new WebSocket("wss://ftx.com/ws/");
  let price = 0;

  onMount(async () => {
    socket.onopen = function (e) {
      console.log("[open] Connection established");
      console.log("Sending request to server");
      socket.send(
        JSON.stringify({
          market: MARKET,
          channel: "ticker",
          op: "subscribe",
        })
      );
    };

    socket.onmessage = function (event) {
      // console.log(`[message] Data received from server: ${event.data}`);
      const eventData = JSON.parse(event.data);

      if (eventData?.type === "update") {
        price = eventData.data.last;
      }
    };

    socket.onclose = function (event) {
      if (event.wasClean) {
        console.log(
          `[close] Connection closed cleanly, code=${event.code} reason=${event.reason}`
        );
      } else {
        // e.g. server process killed or network down
        // event.code is usually 1006 in this case
        console.log("[close] Connection died");
      }
    };

    socket.onerror = function (error) {
      console.error(`[error] ${error.message}`);
    };
  });

  onDestroy(async () => {
    // socket.close(1000, "Work complete");
  });
</script>

<div class="App">
  <div class="Container">
    <div><a href="https://ftx.com/trade/{MARKET}">{MARKET}</a></div>
    <div>${price}</div>
  </div>
</div>

<style>
  a {
    text-decoration: none;
    color: black;
  }

  .App {
    font-family: "Courier New", Courier, monospace;
    font-size: 30px;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
  }

  .Container {
    text-align: center;
  }
</style>
