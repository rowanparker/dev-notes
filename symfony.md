To access the current environment, use the Kernel Interface:

    private KernelInterface $kernel;

    public function __construct(KernelInterface $kernel)
    {
        $this->kernel = $kernel;
    }
    
    //...
    $kernel->getEnvironment());
    //...
