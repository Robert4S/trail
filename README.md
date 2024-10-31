# Trail 

Trail is a minimalistic, composable framework for building HTTP/WebSocket
servers, inspired by [Plug][plug] & [WebSock][websock]. It provides its users
with a small set of abstractions for building _trails_ that can be assembled to
handle a request.

To create a Trail, you can use the syntax `Trail.[fn1;fn2;fn3;...]`, where each
function takes a connection object and an arbitrary context, to produce a new
connection object.

For example:

```ocaml
open Riot
open Trail

module My_handler = struct
  include Sock.Default

  type args = unit
  type state = unit

  let init () = `ok ()

  let handle_frame frame _conn state =
    Riot.Logger.info (fun f -> f "frame: %a" Frame.pp frame);
    `push ([], state)
end

let endpoint =
  [
    use (module Logger) Logger.(args ~level:Debug ());
    router
      [
        socket "/ws" (module My_handler) ();
        get "/" (fun conn -> Conn.send_response `OK {%b|"hello world"|} conn);
        scope "/api"
          [ get "/version" (fun conn -> Conn.send_response `OK {%b|"none"|} conn) ];
      ];
  ]

let start_endpoint_link () =
  let handler = Nomad.trail endpoint in
  Nomad.start_link ~port:8000 ~handler ()

let start () =
  let level = Riot.Logger.Debug in
  Riot.Logger.set_log_level (Some level);
  Supervisor.start_link
    ~child_specs:[ Supervisor.child_spec start_endpoint_link () ]
    ()
```

[riot]: https://github.com/leostera/riot
[plug]: https://hexdocs.pm/plug/readme.html
