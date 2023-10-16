# RVO
In the context of the C++ programming language, [return value optimization](https://en.wikipedia.org/wiki/Copy_elision#:~:text=In%20the%20context%20of%20the,program%20by%20the%20C%2B%2B%20standard) (RVO) is a compiler optimization that involves eliminating the temporary object created to hold a function's return value. RVO is allowed to change the observable behaviour of the resulting program by the C++ standard.
## Code
```C++
#include <vector>
#include <iostream>

void squaresByReference(std::vector<int> &squares){

    for(int i = 0; i < squares.size(); i++)
        squares[i] = (i + 1) * (i + 1);

}

std::vector<int> squaresByValue(){
    std::vector<int> squares(5);

    for(int i = 0; i < squares.size(); i++)
        squares[i] = (i + 1) * (i + 1);

    return squares;
}

int main() {

    std::vector<int> squares = squaresByValue();
    squaresByReference(squares);

    for(int i = 0; i < squares.size(); i++)
        std::cout << squares[i] << std::endl;

    return 0;

}

```

```asm

main():

        ; function prologue
	push   %rbp                     
	push   %rbx                     
	sub    $0x48,%rsp               
	lea    0x40(%rsp),%rbp    
	
	; call to main function
	call   0x7ff6a40918e7 <__main>
	
	; squaresByValue() call
	lea    -0x20(%rbp),%rax                         ; calculates effective address of squares and stores in %rax
	mov    %rax,%rcx                                ; copies value from %rax to %rcx
	call   0x7ff6a4091646 <squaresByValue()>
	
	; At this point check the squaresByValue() function
	
	; squaresByReference() call
	lea    -0x20(%rbp),%rax                         ; calculates effective address of squares and stores in %rax
	mov    %rax,%rcx                                ; copies value from %rax to %rcx
	call   0x7ff6a40915e0 <squaresByReference(std::vector<int, std::allocator<int> >&)>
	
	; At this point check the squaresByReference() function
	
	; std::cout for loop
	movl   $0x0,-0x4(%rbp)          
	jmp    0x7ff6a4091766 <main()+108>
	mov    -0x4(%rbp),%eax          
	movslq %eax,%rdx                
	lea    -0x20(%rbp),%rax         
	mov    %rax,%rcx                
	call   0x7ff6a4093640 <std::vector<int, std::allocator<int> >::operator[](unsigned long long)>
	mov    (%rax),%eax              
	mov    %eax,%edx                
	mov    0x3c38(%rip),%rax        # 0x7ff6a4095380 <__fu0__ZSt4cout>
	mov    %rax,%rcx                
	call   0x7ff6a4091818 <std::basic_ostream<char, std::char_traits<char> >::operator<<(int)>
	mov    %rax,%rcx                
	mov    0x3c36(%rip),%rax        # 0x7ff6a4095390 <.refptr._ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_>
	mov    %rax,%rdx                
	call   0x7ff6a4091820 <std::basic_ostream<char, std::char_traits<char> >::operator<<(std::basic_ostream<char, std::char_traits<char> >& (*)(std::basic_ostream<char, std::char_traits<char> >&))>
	addl   $0x1,-0x4(%rbp)          
	mov    -0x4(%rbp),%eax          
	movslq %eax,%rbx                
	lea    -0x20(%rbp),%rax         
	mov    %rax,%rcx                
	call   0x7ff6a4093040 <std::vector<int, std::allocator<int> >::size() const>
	cmp    %rax,%rbx                
	setb   %al                      
	test   %al,%al                                                                  ; test for loop condition 
	jne    0x7ff6a409172b <main()+49>                                               ; jump back to start of for loop if condition is still true
	
	; allocate space for the 0 from return 0; 
	mov    $0x0,%ebx                
	
	; local main vector is destroyed
	lea    -0x20(%rbp),%rax         
	mov    %rax,%rcx                
	call   0x7ff6a40935e0 <std::vector<int, std::allocator<int> >::~vector()>       ; first destructor call
	mov    %ebx,%eax                
	jmp    0x7ff6a40917b1 <main()+183>                                              ; jump to function epilogue
	
	/*  When any function ends, including main, the local variables gets destroyed, in this case the squares vector variable.
        In addition, when main ends, so does the program,
        thus a second vector destructor is called during the global cleanup process.
        However, it seems the compiler optimizes this by jumping to the function epilogue, skipping the second destructor call.
        */
	
	mov    %rax,%rbx                
	lea    -0x20(%rbp),%rax         
	mov    %rax,%rcx                
	call   0x7ff6a40935e0 <std::vector<int, std::allocator<int> >::~vector()>       ; second destructor call (skipped)
	mov    %rbx,%rax                
	mov    %rax,%rcx                
	call   0x7ff6a4092e80 <_Unwind_Resume>
	
	; function epilogue
	add    $0x48,%rsp               
	pop    %rbx                     
	pop    %rbp                     
	ret                             


```


## squaresByValue() function
```C++
std::vector<int> squaresByValue(){
    std::vector<int> squares(5);

    for(int i = 0; i < squares.size(); i++)
        squares[i] = (i + 1) * (i + 1);

    return squares;
}
```

```asm
squaresByValue():

        ; function prologue
	push   %rbp                     
	push   %rbx                     
	sub    $0x48,%rsp               
	lea    0x40(%rsp),%rbp     
	
	mov    %rcx,0x20(%rbp)                           ; copies value of %rcx to 0x20(%rbp). %rcx is the squares vector address from main
	lea    -0x11(%rbp),%rax   
	mov    %rax,-0x10(%rbp)         
	nop                             
	nop       
	         
	; local vector creation             
	lea    -0x11(%rbp),%rdx         
	mov    0x20(%rbp),%rax          
	mov    %rdx,%r8                 
	mov    $0x5,%edx            ; size 0x5 -> 5         
	mov    %rax,%rcx                
	call   0x7ff69c003800 <std::vector<int, std::allocator<int> >::vector(unsigned long long, std::allocator<int> const&)>
	lea    -0x11(%rbp),%rax         
	mov    %rax,%rcx                
	call   0x7ff69c003580 <std::__new_allocator<int>::~__new_allocator()>
	nop                         
	
	; for loop 
	movl   $0x0,-0x4(%rbp)          
	jmp    0x7ff69c0016b6 <squaresByValue()+112>
	mov    -0x4(%rbp),%eax          
	lea    0x1(%rax),%edx           
	mov    -0x4(%rbp),%eax          
	add    $0x1,%eax                
	mov    %edx,%ebx                
	imul   %eax,%ebx                
	mov    -0x4(%rbp),%eax          
	movslq %eax,%rdx                
	mov    0x20(%rbp),%rax          
	mov    %rax,%rcx                
	call   0x7ff69c003920 <std::vector<int, std::allocator<int> >::operator[](unsigned long long)>
	mov    %ebx,(%rax)              
	addl   $0x1,-0x4(%rbp)          
	mov    -0x4(%rbp),%eax          
	movslq %eax,%rbx                
	mov    0x20(%rbp),%rax          
	mov    %rax,%rcx                
	call   0x7ff69c003100 <std::vector<int, std::allocator<int> >::size() const>
	cmp    %rax,%rbx                
	setb   %al                      
	test   %al,%al                                                          ; test for loop condition
	jne    0x7ff69c00168d <squaresByValue()+71>                             ; jump back to start of for loop if condition is still true
	
	; in place of return statement
	jmp    0x7ff69c0016ef <squaresByValue()+169>                            ; jumps to move    0x20(%rbp),%rax
	
	mov    %rax,%rbx                
	lea    -0x11(%rbp),%rax         
	mov    %rax,%rcx                
	call   0x7ff69c003580 <std::__new_allocator<int>::~__new_allocator()>   ; likely local vector's allocator destructor (skipped)
	nop                             
	mov    %rbx,%rax                
	mov    %rax,%rcx                
	call   0x7ff69c002ee0 <_Unwind_Resume>
	
	; function epilogue
	mov    0x20(%rbp),%rax                                                  ; jumps here, fails to destruct local vector
	add    $0x48,%rsp               
	pop    %rbx                     
	pop    %rbp                     
	ret   
```

## squaresByReference() function
```C++
void squaresByReference(std::vector<int> &squares){

    for(int i = 0; i < squares.size(); i++)
        squares[i] = (i + 1) * (i + 1);

}
```

```asm
squaresByReference(std::vector<int, std::allocator<int> >&):
    ; function prologue
	push   %rbp                     
	push   %rbx                     
	sub    $0x38,%rsp               
	lea    0x30(%rsp),%rbp  
	        
	mov    %rcx,0x20(%rbp)          ; copies value of %rcx to 0x20(%rbp). %rcx is the squares vector address from main
	
	; for loop
	movl   $0x0,-0x4(%rbp)          
	jmp    0x7ff7a28e1621 <squaresByReference(std::vector<int, std::allocator<int> >&)+65>
	mov    -0x4(%rbp),%eax          
	lea    0x1(%rax),%edx           
	mov    -0x4(%rbp),%eax          
	add    $0x1,%eax                
	mov    %edx,%ebx                
	imul   %eax,%ebx                
	mov    -0x4(%rbp),%eax          
	movslq %eax,%rdx                
	mov    0x20(%rbp),%rax          
	mov    %rax,%rcx                
	call   0x7ff7a28e3920 <std::vector<int, std::allocator<int> >::operator[](unsigned long long)>
	mov    %ebx,(%rax)              
	addl   $0x1,-0x4(%rbp)          
	mov    -0x4(%rbp),%eax          
	movslq %eax,%rbx                
	mov    0x20(%rbp),%rax          
	mov    %rax,%rcx                
	call   0x7ff7a28e3100 <std::vector<int, std::allocator<int> >::size() const>
	cmp    %rax,%rbx                
	setb   %al                      
	test   %al,%al                                                                              ; test for loop condition  
	jne    0x7ff7a28e15f8 <squaresByReference(std::vector<int, std::allocator<int> >&)+24>      ; jump back to start of for loop if condition is still true
	
	; function epilogue
	nop                             
	nop                             
	add    $0x38,%rsp               
	pop    %rbx                     
	pop    %rbp                     
	ret                             
```

With the premise that all of my assumptions and interpretations are correct, the compiler does perform Return Value Optimization (RVO) for the `squaresByValue()` function, avoiding the copy constructor call. There is almost no difference between returning the vector by value (since the compiler optimizes it) and passing it by reference. The only main difference is that in `squaresByValue()` the function does have to create the local vector, such that the effective memory address we stored in `%rax` is the place in memory where this local vector is stored. We can see that the program fails to destroy this vector, and thus fails to free its memory, by skipping the destructor call with a `jump` statement, meaning that our `squares` vector in main will now point to an initialized vector.
On the other hand, when calling `squaresByReference()` we can see that the program executes the same instructions it did for `squaresByValue()`, further proving that the difference between these two is negligible. Of course in this case there is already memory allocated for `squares` so there is no more vector initialization.
