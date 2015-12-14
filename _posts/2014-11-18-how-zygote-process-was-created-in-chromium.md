---
layout: post
title: How zygote and render process was created in Chromium
tags: chromium zygote
categories: chromium
---


How zygote process was created


void BrowserMainLoop::EarlyInitialization


void SetupSandbox(const base::CommandLine& parsed_command_line) 


ZygoteHostImpl::GetInstance()->Init(sandbox_binary.value());




void ZygoteHostImpl::Init(const std::string& sandbox_cmd)  {


base::FilePath chrome_path;
CHECK(PathService::Get(base::FILE_EXE, &chrome_path));
cmd_line.AppendSwitchASCII(switches::kProcessType, switches::kZygoteProcess);
base::CommandLine cmd_line(chrome_path);
base::LaunchProcess(cmd_line.argv(), options, &process);
}


void ContentMainDelegateEfl::PreSandboxStartup()  
PathService::Override(base::FILE_EXE, base::FilePath(SubProcessPath()));


std::string SubProcessPath() {
pak_file = pak_dir.Append(FILE_PATH_LITERAL("efl_webprocess"));



src/base/process/launch_posix.cc
base::LaunchProcess   
pid_t pid;
#if defined(OS_LINUX)
if (options.clone_flags) {
// Signal handling in this function assumes the creation of a new
// process, so we check that a thread is not being created by mistake
// and that signal handling follows the process-creation rules.
RAW_CHECK(
!(options.clone_flags & (CLONE_SIGHAND | CLONE_THREAD | CLONE_VM)));
pid = syscall(__NR_clone, options.clone_flags, 0, 0, 0); 
} else
#endif
{
pid = fork();   // here fork the zygote process in browser process, 
  // so browser is parent process of zygote process 
} 
 
for (size_t i = 0; i < argv.size(); i++)
  argv_cstr[i] = const_cast<char*>(argv[i].c_str());
argv_cstr[argv.size()] = NULL;


// create efl_webprocess with zygote type
execvp(argv_cstr[0], argv_cstr.get());  // the first args is “/usr/bin/efl_webprocess”
int execvp(const char *file, char const argv[]);
When execvp() is executed, the program file given by the first argument will be loaded into the caller's address space and over-write the program there. 


Then, the second argument will be provided to the program and starts the execution. 


As a result, once the specified program file starts its execution, the original program in the caller's address space is gone and is replaced by the new program.
efl_webprocess
impl/web_process.cc 
int main(int argc, const char **argv) {
  LOG(INFO) << "web process launching...";


  content::WebProcessContentMainDelegateEfl content_main_delegate;
  content::ContentMainParams params(&content_main_delegate);


  params.argc = argc;
  params.argv = argv;


  return content::ContentMain(params);
}


int ContentMain(const ContentMainParams& params) {
  scoped_ptr<ContentMainRunner> main_runner(ContentMainRunner::Create());


  int exit_code = main_runner->Initialize(params);
  if (exit_code >= 0)
    return exit_code;


  exit_code = main_runner->Run(); // here begin the main loop of zygote process


  main_runner->Shutdown();


  return exit_code;
}


src/content/app/content_main_runner.cc
int Initialize(const ContentMainParams& params) override {  
   return RunNamedProcessTypeMain(process_type, main_params, delegate_);


src/content/app/content_main_runner.cc
int Initialize(const ContentMainParams& params) override {  
   return RunNamedProcessTypeMain(process_type, main_params, delegate_);   




// Run the FooMain() for a given process type.                                                                                                                     
// If |process_type| is empty, runs BrowserMain().                                                                                                                  
// Returns the exit code for this process.                                                                                                                           
int RunNamedProcessTypeMain(                                                                                                                                         
    const std::string& process_type,                                                                                                                                 
    const MainFunctionParams& main_function_params,                                                                                                                 
    ContentMainDelegate* delegate) {         

// here if create a new render process, it will go to RendererMain directly
#if defined(OS_POSIX) && !defined(OS_MACOSX) && !defined(OS_ANDROID)                                                                                                              
  // Zygote startup is special -- see RunZygote comments above                                                                                                                     
  // for why we don't use ZygoteMain directly.                                                                                                                                   
  if (process_type == switches::kZygoteProcess)                                                                                                                                         
    return RunZygote(main_function_params, delegate);                                                                                                                            
#endif       


int RunZygote(const MainFunctionParams& main_function_params,                                                                                                      
              ContentMainDelegate* delegate) {             
 
   // This function call can return multiple times, once per fork().                                                                                  
  if (!ZygoteMain(main_function_params, zygote_fork_delegates.Pass()))                                                                                             
    return 1;       
~        
bool ZygoteMain(const MainFunctionParams& params,                                                                                                                  
                ScopedVector<ZygoteForkDelegate> fork_delegates) {           
 Zygote zygote(sandbox_flags, fork_delegates.Pass(), extra_children,                                                                                          
                extra_fds);                                                                                                                                        
  // This function call can return multiple times, once per fork().                                                                                                
  return zygote.ProcessRequests();  
  


bool Zygote::ProcessRequests() {  
  for (;;) {                                                                                                                                                                    
    // This function call can return multiple times, once per fork().                                                                                              
    if (HandleRequestFromBrowser(kZygoteSocketPairFd))                                                                                                             
      return true;
How render process was created
--single-process
// set single process model


BrowserMainLoop::PreCreateThreads
if (parsed_command_line_.HasSwitch(
switches::kSingleProcess))
RenderProcessHost::SetRunRendererInProcess(true);


content::RenderProcessHostImpl::Init
    // Setup the IPC channel.                                                                                                                                                     
    const std::string channel_id =                                                                                                                                               
       IPC::Channel::GenerateVerifiedChannelID(std::string());                                                                                                                  
    channel_ = CreateChannelProxy(channel_id);   

// Setup the Mojo channel. 
mojo_application_host_->Init();      


CreateMessageFilters(); 
if (run_renderer_in_process()) { 
in_process_renderer_.reset(g_renderer_main_thread_factory(channel_id));


src/content/browser/renderer_host/render_process_host_impl.cc

void RenderProcessHostImpl::RegisterRendererMainThreadFactory(                                                                                                       
    RendererMainThreadFactoryFunction create) {                                                                                                                     
  g_renderer_main_thread_factory = create;                                                                                                                          
}


impl/ewk_global_data.cc
void EwkGlobalData::Ensure() { 
   content::RenderProcessHostImpl::RegisterRendererMainThreadFactory(                                                                                                         
        content::CreateInProcessRendererThread); 


base::Thread* CreateInProcessRendererThread(const std::string& channel_id) {                                                                                                           
  return new InProcessRendererThread(channel_id);                                                                                                                                  
} 
InProcessRendererThread
./src/content/renderer/in_process_renderer_thread.cc 
InProcessRendererThread::InProcessRendererThread(const std::string& channel_id)
: Thread("Chrome_InProcRendererThread"), channel_id_(channel_id) {
}


InProcessRendererThread::~InProcessRendererThread() {
  Stop();
}


void InProcessRendererThread::Init() {
  // here create a new render process in renderer thread
  render_process_.reset(new RenderProcessImpl());
  // here begin a render  thread
  new RenderThreadImpl(channel_id_);
}




How render process was created
content::RenderProcessHostImpl::Init


      if (run_renderer_in_process()) { 
} else {
// Build command line for renderer.  We call AppendRendererCommandLine()
// first so the process type argument will appear first.
base::CommandLine* cmd_line = new base::CommandLine(renderer_path);
if (!renderer_prefix.empty())
 cmd_line->PrependWrapper(renderer_prefix);
AppendRendererCommandLine(cmd_line);
cmd_line->AppendSwitchASCII(switches::kProcessChannelID, channel_id);


// Spawn the child process asynchronously to avoid blocking the UI thread.
// As long as there's no renderer prefix, we can use the zygote process
// at this stage.
child_process_launcher_.reset(new ChildProcessLauncher(
new RendererSandboxedProcessLauncherDelegate(channel_.get()),
cmd_line,
GetID(),
this));


fast_shutdown_started_ = false;

}
void RenderProcessHostImpl::AppendRendererCommandLine(
    base::CommandLine* command_line) const {
  // Pass the process type first, so it shows first in process listings.
  command_line->AppendSwitchASCII(switches::kProcessType,
                                  switches::kRendererProcess);

ChildProcessLauncher
ChildProcessLauncher::ChildProcessLauncher(
    SandboxedProcessLauncherDelegate* delegate,
    base::CommandLine* cmd_line,
    int child_process_id,
    Client* client) {
  context_ = new Context();
  context_->Launch(
      delegate,
      cmd_line,
      child_process_id,
      client);
}   
 ChildProcessLauncher::Context::Launch(SandboxedProcessLauncherDelegate* delegate,
              base::CommandLine* cmd_line,
              int child_process_id,
              Client* client) {
    client_ = client;


    CHECK(BrowserThread::GetCurrentThreadIdentifier(&client_thread_id_));
    BrowserThread::PostTask(
        BrowserThread::PROCESS_LAUNCHER, FROM_HERE,
        base::Bind(&Context::LaunchInternal,
                   make_scoped_refptr(this),
                   client_thread_id_,
                   child_process_id,
                   delegate,
                   cmd_line));
  }


 ./src/content/browser/child_process_launcher.cc
  ChildProcessLauncher::Context::LaunchInternal(
bool use_zygote = delegate->ShouldUseZygote();
base::ProcessHandle handle = base::kNullProcessHandle;
     if (use_zygote) {
      handle = ZygoteHostImpl::GetInstance()->ForkRequest(
          cmd_line->argv(), files_to_register.Pass(), process_type);
}
     else {
bool launched = base::LaunchProcess(*cmd_line, options, &handle);
     }


 src/content/public/common/content_switches.cc
  const char kRendererCmdPrefix[]             = "renderer-cmd-prefix"; 
  
  src/content/browser/renderer_host/render_process_host_impl.cc
  bool ShouldUseZygote() override {
    const base::CommandLine& browser_command_line =
        *base::CommandLine::ForCurrentProcess();
    base::CommandLine::StringType renderer_prefix =
        browser_command_line.GetSwitchValueNative(switches::kRendererCmdPrefix);
    return renderer_prefix.empty();
  }
ZygoteHostImpl::ForkRequest
pid_t ZygoteHostImpl::ForkRequest(const std::vector<std::string>& argv,
                                  scoped_ptr<FileDescriptorInfo> mapping,
                                  const std::string& process_type) {
  DCHECK(init_);
  Pickle pickle;
    
  int raw_socks[2];
  PCHECK(0 == socketpair(AF_UNIX, SOCK_SEQPACKET, 0, raw_socks));
  base::ScopedFD my_sock(raw_socks[0]);
  base::ScopedFD peer_sock(raw_socks[1]);
  CHECK(UnixDomainSocket::EnableReceiveProcessId(my_sock.get()));
  
  pickle.WriteInt(kZygoteCommandFork);
  pickle.WriteString(process_type);
  pickle.WriteInt(argv.size());
  for (std::vector<std::string>::const_iterator
       i = argv.begin(); i != argv.end(); ++i)
    pickle.WriteString(*i);


// send socket message to zygote process
Handle Request in Zygote Process
src/content/zygote/zygote_linux.cc 
bool Zygote::HandleRequestFromBrowser(int fd) {
 switch (kind) {
      case kZygoteCommandFork:
        // This function call can return multiple times, once per fork().
        return HandleForkRequest(fd, pickle, iter, fds.Pass());
bool Zygote::HandleForkRequest(int fd,                                                                                                                                               
                               const Pickle& pickle,                                                                                                                                 
                               PickleIterator iter,                                                                                                                                  
                               ScopedVector<base::ScopedFD> fds) {                                                                                                                   
base::ProcessId child_pid = ReadArgsAndFork( 
 if (child_pid == 0)
    return true;  // in the new fork process, render process, it return true here
base::ProcessId Zygote::ReadArgsAndFork(const Pickle& pickle,
   // Returns twice, once per process.
  base::ProcessId child_pid = ForkWithRealPid(process_type,
                                              mapping,
                                              channel_id,
                                              pid_oracle.Pass(),
                                              uma_name,
                                              uma_sample,
                                              uma_boundary_value);
Zygote::ForkWithRealPid
int Zygote::ForkWithRealPid(const std::string& process_type,
                            const base::GlobalDescriptors::Mapping& fd_mapping,
                            const std::string& channel_id,
                            base::ScopedFD pid_oracle,
                            std::string* uma_name,
                            int* uma_sample,
                            int* uma_boundary_value) {
 
 
  base::ProcessId pid = 0;
  if (helper) {
    int ipc_channel_fd = LookUpFd(fd_mapping, kPrimaryIPCChannel);
    if (ipc_channel_fd < 0) {
      DLOG(ERROR) << "Failed to find kPrimaryIPCChannel in FD mapping";
      return -1;
    }
    std::vector<int> fds;
    fds.push_back(ipc_channel_fd);  // kBrowserFDIndex
    fds.push_back(pid_oracle.get());  // kPIDOracleFDIndex
    pid = helper->Fork(process_type, fds, channel_id);


    // Helpers should never return in the child process.
    CHECK_NE(pid, 0);
  } else {
    CreatePipe(&read_pipe, &write_pipe);
    pid = fork();
  }


// here fork a new process in zygote process, so zygote process is the parent process of render
Handle Request in Render Process
bool Zygote::HandleForkRequest(int fd,  {
  base::ProcessId child_pid = ReadArgsAndFork( 
if (child_pid == 0)
   return true;  // in the new fork process, render process, it return true here
retrun false; // in the parent process, zygote process, it return false here
}


bool Zygote::HandleRequestFromBrowser
  
      switch (kind) {
      case kZygoteCommandFork:
        // This function call can return multiple times, once per fork().
        return HandleForkRequest(fd, pickle, iter, fds.Pass());


src/content/zygote/zygote_linux.cc
 bool Zygote::ProcessRequests() {
 
  for (;;) {
    // This function call can return multiple times, once per fork().
    if (HandleRequestFromBrowser(kZygoteSocketPairFd))  // in the parent process, zygote process, it still in for(;;) circle because former false return
      return true;   // in the new fork process, render process, it will return true here and break the for (;;) circle
  }
}
src/content/zygote/zygote_main_linux.cc
bool ZygoteMain(const MainFunctionParams& params,
                ScopedVector<ZygoteForkDelegate> fork_delegates) {
   // This function call can return multiple times, once per fork().
  return zygote.ProcessRequests();
}
After fork in Zygote Process 
int RunZygote(const MainFunctionParams& main_function_params,
              ContentMainDelegate* delegate) {
  static const MainFunction kMainFunctions[] = {
    { switches::kRendererProcess,    RendererMain },
#if defined(ENABLE_PLUGINS)
    { switches::kPpapiPluginProcess, PpapiPluginMain },
#endif
    { switches::kUtilityProcess,     UtilityMain },
  };
  
  // This function call can return multiple times, once per fork().
  if (!ZygoteMain(main_function_params, zygote_fork_delegates.Pass()))
    return 1; // for new child process, the render process, ZygoteMain() return true, so go on
// but for zygote prcess, it still in below for(;;) loop, and it goes to HandleRequestFromBrowser 
bool Zygote::ProcessRequests() {
  for (;;) {
    // This function call can return multiple times, once per fork().
    if (HandleRequestFromBrowser(kZygoteSocketPairFd))
      return true;
  }
}


bool Zygote::HandleRequestFromBrowser(int fd) {
  ScopedVector<base::ScopedFD> fds;
  char buf[kZygoteMaxMessageLength];
  const ssize_t len = UnixDomainSocket::RecvMsg(fd, buf, sizeof(buf), &fds);  // here zygote process will wait for the signal from browser process and then handle it




RunZygote continue 
in Render Process
int RunZygote(const MainFunctionParams& main_function_params,
              ContentMainDelegate* delegate) {


 if (delegate) delegate->ZygoteForked();
  


  // Zygote::HandleForkRequest may have reallocated the command
  // line so update it here with the new version.
  const base::CommandLine& command_line =
      *base::CommandLine::ForCurrentProcess();
  std::string process_type =
      command_line.GetSwitchValueASCII(switches::kProcessType);
  ContentClientInitializer::Set(process_type, delegate);
    
  // The StatsTable must be initialized in each process; we already
  // initialized for the browser process, now we need to initialize
  // within the new processes as well.
  InitializeStatsTable(command_line);
                                    
  MainFunctionParams main_params(command_line);
  main_params.zygote_child = true;


  for (size_t i = 0; i < arraysize(kMainFunctions); ++i) {
    if (process_type == kMainFunctions[i].name)
      return kMainFunctions[i].function(main_params); 
  }
RendererMain
src/content/renderer/renderer_main.cc  
// mainline routine for running as the Renderer process
int RendererMain(const MainFunctionParams& parameters) {
base::MessageLoop main_message_loop;


#if defined(OS_POSIX) && !defined(OS_MACOSX)
    RenderProcessImpl render_process;
    new RenderThreadImpl();
#endif


      TRACE_EVENT_BEGIN_ETW("RendererMain.START_MSG_LOOP", 0, 0);
      base::MessageLoop::current()->Run();  // here begin the main loop of render process
      TRACE_EVENT_END_ETW("RendererMain.START_MSG_LOOP", 0, 0);