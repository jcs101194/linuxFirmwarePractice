#include <iostream>
#include <string.h>
#include <fstream>
#include <unistd.h>
#include <ctype.h>
#include <signal.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <errno.h>
#include <fcntl.h>
#include <time.h>


//44 hours
//Every px should be it's own thread
//We can suppose that every px is a process arithmetic "space"
//That consists of only one function and the process is either
//killed or paused if the PC passes the end of the function

using namespace std;

bool testMode = false;
bool parentReady = true;
bool currentChildReadMessage = false;       //This line shouldn't be too dangerous. Every child will be handled in parallel.
bool printGraph = false;
bool verbose = false;
pid_t parentPid = getpid();

void clearArray(char desiredArray[], int numberOfElements);
void arithmeticSpace(int,string,int);
void sigStop(int parameter);
void sigContinue(int, siginfo_t*, void*);
void SIGUSR1Handler(int, siginfo_t*, void*);
void SIGUSR2Handler(int, siginfo_t*, void*);

struct p
{
	string identifier = "-";
    int value = 0;              //pId in this struct's case
    int innerValue = 0;
    bool inUse = false;
    bool paused = true;
    bool writtenBack = false;
    int *personalPipe;
};
struct var
{
	string identifier = "-";
	int value = 0;
	int innerValue = 0;
	bool inUse = false;
	bool paused = true;
	bool writtenBack = false;
	int *personalPipe;
};
struct commandLineArguments
{
	string fileName;
	string numbersFile;
};

template <class T>
class StructArray
{
    T *objectsArray;
    int index;
    int arraySize;
    string genericType;

 public:

    StructArray(string desiredType);
    void addElement(string desiredIdentifier, int desiredValue);
    void printParameters(string *desiredArray);
    void printArrayState();
    void printVariableState();
    void updateIdentifier(string desiredIdentifier, int i){objectsArray[i].identifier = desiredIdentifier;};
    void appendIdentifier(string desiredIdentifier);
    void updateInnerValueByValue(int desiredValue, int desiredInnerValue);
    void updateValue(int desiredValue, int i){objectsArray[i].value = desiredValue;};
    void updateInUse(string desiredIdentifier, bool desiredValue);
    void updateInUseByValue(int desiredValue, bool desiredBoolean);
    void updatePaused(string desiredIdentifier, bool desiredBoolean);
    void updatePausedByValue(int desiredIdentifier, bool desiredBoolean);
    void updateWrittenBack(string desiredIdentifier);
    void updatePipe(string desiredIdentifier);
    void initializeNextPipe();
    T returnElement(string desiredIdentifier);
    T returnElementByIndex(int desiredIndex);
    T returnElementByValue(int desiredValue);
    T popElement();
    int returnArraySize(){return arraySize;};
    int returnValueByIdentifier(string desiredIdentifier);
    string returnIdentifierByValue(pid_t desiredPid);
    bool isInitialized(string desiredString);
    bool isPaused(string desiredIdentifier);
    int* returnFirstUnclaimedPipe();
    int* returnPipe(string desiredIdentifier);
    int returnIndex(string desiredIdentifier);
};


template <class T>
StructArray<T>::StructArray(string desiredType)
{
    index = 0;
    arraySize = 10;
    genericType = desiredType;
    objectsArray = new T[arraySize];
}

template <class T>
void StructArray<T>::addElement(string desiredIdentifier, int desiredValue)
{
    if(index >= 0 && index < 10)
    {
        objectsArray[index].identifier = desiredIdentifier;
        objectsArray[index].value = desiredValue;
        index++;
    }
}
template <class T>
void StructArray<T>::appendIdentifier(string desiredIdentifier)
{
    //Note that "-" means the identifier slot is "empty"
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].identifier == "-")
        {
            objectsArray[i].identifier = desiredIdentifier;
            return;
        }
}
template <class T>
void StructArray<T>::printParameters(string *desiredArray)
{
    /*
        print like:

        a = 3
        b = 7
        p0 = 12
    */

    //If element in objectsArray is in desiredArray then print that element
    for(int i = 0; i < arraySize; i++)
    {
        for(int j = 0; j < sizeof(desiredArray) && desiredArray[j] != ""; j++)
            if(objectsArray[i].identifier == desiredArray[j])
            {
                if(genericType == "var")
                    cout << objectsArray[i].identifier << " = " << objectsArray[i].value << endl;
                if(genericType == "p")
                    cout << objectsArray[i].identifier << " = " << objectsArray[i].innerValue  << endl;

                //objectsArray[i].identifier = "-";
                objectsArray[i].value = 0;
                break;
            }
    }
}
template <class T>
void StructArray<T>::printArrayState()
{
    if(genericType == "p")
    {
        cout << "i identifier\tpId\tinnerValue\tinUse\tpersonalPipe\tpaused\twrittenBack\n";
        for(int i = 0; i < arraySize; i++)
        {
            cout << i << "  "
                 << objectsArray[i].identifier << "\t\t"
                 << objectsArray[i].value << "\t"
                 << objectsArray[i].innerValue << "\t\t"
                 << objectsArray[i].inUse << "\t"
                 << objectsArray[i].personalPipe << "\t"
                 << objectsArray[i].paused <<  "\t"
                 << objectsArray[i].writtenBack << endl;
        }
    }
    if(genericType == "var")
    {
        cout << "i identifier\tvalue\n";
        for(int i = 0; i < arraySize; i++)
        {
            cout << i << "  "
             << objectsArray[i].identifier << "\t\t"
             << objectsArray[i].value << "\t"
             << endl;
        }
    }


}

template <class T>
T StructArray<T>::returnElement(string desiredIdentifier)
{
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].identifier == desiredIdentifier)
            return objectsArray[i];

    return {};
}
template <class T>
T StructArray<T>::returnElementByIndex(int desiredIndex)
{
    T answer;
    if(index >= 0)
    {
        answer = objectsArray[index];
    }

    return answer;
}
template <class T>
T StructArray<T>::returnElementByValue(int desiredValue)
{
    //Sometimes it is necessary to return struct instance by pID
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].value == desiredValue)
            return objectsArray[i];
    return {};
}

template <class T>
T StructArray<T>::popElement()
{
    T desiredElement;

    if(index >= 0)
    {
        desiredElement = objectsArray[index];
        index--;
    }
    else
        desiredElement = NULL;

    return desiredElement;
}
template <class T>
int StructArray<T>::returnValueByIdentifier(string desiredIdentifier)
{
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].identifier == desiredIdentifier)
            return objectsArray[i].value;

    return -2;
}
template <class T>
string StructArray<T>::returnIdentifierByValue(pid_t desiredPid)
{
    //Only for use with internalVariableArray
    string utilityString = to_string(desiredPid);
    int integerValue = stoi(utilityString);
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].value == integerValue)
            return objectsArray[i].identifier;
}
template <class T>
void StructArray<T>::initializeNextPipe()
{
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].personalPipe == 0)
        {
            objectsArray[i].personalPipe = new int[2];
            pipe2(objectsArray[i].personalPipe, O_NONBLOCK);
            return;
        }
}
template <class T>
void StructArray<T>::updateInnerValueByValue(int desiredValue, int desiredInnerValue)
{
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].value == desiredValue)
            objectsArray[i].innerValue = desiredInnerValue;
}
template <class T>
void StructArray<T>::updateInUse(string desiredIdentifier, bool desiredValue)
{
    //Specifically, for use for internal variables
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].identifier == desiredIdentifier)
        {
            objectsArray[i].inUse = desiredValue;
        }
}
template <class T>
void StructArray<T>::updateInUseByValue(int desiredValue, bool desiredBoolean)
{
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].value == desiredValue)
            objectsArray[i].inUse = desiredBoolean;
}
template <class T>
void StructArray<T>::updatePaused(string desiredIdentifier, bool desiredBoolean)
{
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].identifier == desiredIdentifier)
        {
            objectsArray[i].paused = desiredBoolean;
            objectsArray[i].inUse = !desiredBoolean;
        }
}
template <class T>
void StructArray<T>::updatePausedByValue(int desiredValue, bool desiredBoolean)
{
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].value == desiredValue)
        {
            objectsArray[i].paused = desiredBoolean;
            objectsArray[i].inUse = !desiredBoolean;
        }
}
template <class T>
void StructArray<T>::updatePipe(string desiredIdentifier)
{
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].identifier == desiredIdentifier)
        {
            objectsArray[i].personalPipe = new int[2];
            pipe(objectsArray[i].personalPipe);
        }
}
template <class T>
void StructArray<T>::updateWrittenBack(string desiredIdentifier)
{
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].identifier == desiredIdentifier)
            objectsArray[i].writtenBack = true;
}
template <class T>
bool StructArray<T>::isPaused(string desiredIdentifier)
{
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].identifier == desiredIdentifier)
            if(objectsArray[i].paused)
                return true;
            else
                return false;


}
template <class T>
bool StructArray<T>::isInitialized(string desiredString)
{
    //The point of this function is to not create unecessary processes
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].identifier == desiredString)
            if(objectsArray[i].value != 0)
            {
                return true;
            }
            else
            {
                return false;
            }
}
template <class T>
int* StructArray<T>::returnFirstUnclaimedPipe()
{
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].value == 0)
            return objectsArray[i].personalPipe;
}
template <class T>
int* StructArray<T>::returnPipe(string desiredIdentifier)
{
    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].identifier == desiredIdentifier)
            return objectsArray[i].personalPipe;

    return NULL;
}

template <class T>
int StructArray<T>::returnIndex(string desiredIdentifier)
{
    //If identifier is not yet stored then return -1.

    for(int i = 0; i < arraySize; i++)
        if(objectsArray[i].identifier == desiredIdentifier)
            return i;

    return -1;
}
//~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~class functions implementation~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~//

static StructArray<p> internalVariableArray("p");
static StructArray<var> inputVariableArray("var");

bool isOperator(char desiredCharacter)
{
	if (desiredCharacter == '+' || desiredCharacter == '-' ||
	    desiredCharacter == '*' || desiredCharacter == '/')
        return true;
	else
		return false;
}
void parseCommandLineArguments(commandLineArguments &desiredStruct, char **argv)
{
	//Let's say, the command line argument is structured like
	//so: "input=1.txt;numbers=2.txt"

	const int arraySize = 40;
	const int tokenArraySize = 20;
	string desiredLine = argv[1];
	static char stringArray[arraySize];
	bool equalSignPassed = false;
	static char tokenArray[tokenArraySize];
	char currentCharacter;
	int j = 0, argumentNumber = 0;

	desiredLine.copy(stringArray, desiredLine.length());

	for (int i = 0; i < arraySize; i++)
	{
		currentCharacter = stringArray[i];
		if(currentCharacter == ';')
		{
			desiredStruct.fileName = (string) tokenArray;
			clearArray(tokenArray, (sizeof(tokenArray)/sizeof(*tokenArray)));
			equalSignPassed = false;

			j = 0;
		}

		if(equalSignPassed)
		{
			tokenArray[j] = currentCharacter;
			j++;
		}
		if(currentCharacter == '=')
			equalSignPassed = true;
	}

	desiredStruct.numbersFile = (string) tokenArray;

}
void clearArray(char desiredArray[], int numberOfElements)
{
	for (int i = 0; i < numberOfElements; i++)
		desiredArray[i] =  '\0';

}

void tokenizeNumbers(string desiredLine)
{
    /*I believe the implication is that the order of the numbers
      corresponds to the way the input_var are declared.

      Another implication is that desiredLine contains only ints.
      */
      const int maxSize = 40;
      static char stringArray[maxSize];
      static char tokenArray[maxSize];
      desiredLine.copy(stringArray, desiredLine.length());
      int j = 0, k = 0;
      char currentCharacter;

      for (int i = 0; i < maxSize; i++)
      {
            currentCharacter = stringArray[i];
            if(currentCharacter != ' ' && currentCharacter != ',' && currentCharacter != '\0')
            {
                tokenArray[j] = currentCharacter;
                j++;
            }
            if (currentCharacter == ' ' || currentCharacter == ',' || currentCharacter == '\0')
            {

                inputVariableArray.addElement("-", atoi( tokenArray));
                //if(testMode) inputVariableArray.printArrayState();
                k++;
                j=0;
                clearArray(tokenArray, (sizeof(tokenArray)/sizeof(*tokenArray)));
            }

            //The exit should be after the currentCharacter is read
            if(currentCharacter == '\0')
                break;

      }

}
bool isInternalVariable(string desiredString)
{
    if(desiredString.at(0) == 'p')
        return true;

    return false;
}
bool isDelimiter(char desiredCharacter)
{
    if(desiredCharacter == ' '  || desiredCharacter == ',' ||
       desiredCharacter == '\0' || desiredCharacter == ';' ||
       desiredCharacter == '('  || desiredCharacter == ')')
        return true;
    else
        return false;
}
void printCharacterArray(char *desiredArray, int arraySize)
{
    for(int i =0; i < arraySize; i++)
        cout << desiredArray[i];

    cout << endl;
}
void tokenizeMessage(char *desiredMessage, string &desiredOperator, string &operandOne)
{
    const int arraySize = 40;
    static char tokenArray[arraySize];
    int j = 0, k = 0;

    for(int i = 0; i < arraySize; i++)
    {
        if(!isDelimiter(desiredMessage[i]))
        {
            tokenArray[j] = desiredMessage[i];
            j++;
        }
        else
        {
            if(k == 0)
                desiredOperator = (string) tokenArray;
            if(k == 1)
                operandOne = (string) tokenArray;

            clearArray(tokenArray, arraySize);
            j = 0;
            k++;
        }
    }
}
void customPause()
{
    //This function will pause and inform its parent
    //Children processes never know its own personal pipe
    //This function will only be used on children processes
    int *desiredPipe = internalVariableArray.returnFirstUnclaimedPipe();
    int utilityInteger;
    kill(getppid(), SIGCONT);
    if(testMode)
        cout << getpid() << " has custom paused." << endl;

    pause();
}
void tokenizeMessageForParent(char messageArray[], int messageSize,string desiredArray[])
{
    //Precondition: messageArray always end with an ';'
    const int tokenArraySize = 20;
    static char tokenArray[tokenArraySize];
    int j = 0, k = 0;

    for(int i = 0; i < messageSize; i++)
    {
        if(messageArray[i] == ';')
        {
            desiredArray[k] = (string) tokenArray;
            k++;

            clearArray(tokenArray, tokenArraySize);
            j = 0;
        }
        else
        {
            tokenArray[j] = messageArray[i];
            j++;
        }
    }
}
void handleMessageForParent(string desiredMessage)
{
    //Note that inUse and notInUse will never be called.
    //only paused or unpaused, but both will be modified accordingly.
    const int messageArraySize = 40, stringArraySize = 10;
    static char messageArray[messageArraySize];
    string utilityString, desiredPid;
    int utilityInteger = 0, equalsIndex = 0;
    static string stringArray[stringArraySize];

    desiredMessage.copy(messageArray, desiredMessage.length());
    tokenizeMessageForParent(messageArray, desiredMessage.length()+1, stringArray);

    for(int i = 0; i < stringArraySize && stringArray[i] != ""; i++)
    {
        desiredMessage = stringArray[i];
        if(testMode) cout << "Parent just read <" << desiredMessage << ">, and parsed it." << endl;
        if(string::npos !=  desiredMessage.find("paused"))
        {
            utilityString = desiredMessage.substr(0,desiredMessage.find("paused")-1);
            utilityInteger = stoi(utilityString);
            internalVariableArray.updatePausedByValue(utilityInteger,true);
            internalVariableArray.updateInUseByValue(utilityInteger,false);
        }
        else if(string::npos != desiredMessage.find("unPaused"))
        {
            utilityString = desiredMessage.substr(0,desiredMessage.find("unPaused")-1);
            utilityInteger = stoi(utilityString);
            internalVariableArray.updatePausedByValue(utilityInteger,false);
            internalVariableArray.updateInUseByValue(utilityInteger, true);
        }
        else
        {
            //The first operand is a number as well as the second operand. That is,
            //the first number is the pid and the second the innerValue
            desiredPid = desiredMessage.substr(0,desiredMessage.find("="));
            utilityInteger = stoi(desiredMessage.substr(desiredMessage.find("=")+1,desiredMessage.length()-1));
            internalVariableArray.updateInnerValueByValue(stoi(desiredPid), utilityInteger);

        }

        if(testMode && verbose)
            internalVariableArray.printArrayState();
    }

}
void informParent(string desiredMessage, int* desiredPipe)
{
    //This function should only be called iff this pid and its pipe are initialized.
    //It should. The child process's internalVariableArray will never be updated,
    //therefore the desiredPipe will always be "unclaimed"
    int utilityInteger;
    utilityInteger = write(desiredPipe[1],desiredMessage.c_str(),desiredMessage.length());
    if(testMode)
        cout << "The message <" << desiredMessage << "> was sent to the parent process from child "
             << getpid() << " with pipe " << desiredPipe << endl;

}
void arithmeticSpace(string firstOperand, string currentOperator, string secondOperand)
{
    /*
        This is where the internal variables are solved.
        In every iteration the loop is paused and updated
        by results with a pipe. When the math is done, the process
        is terminated before it leaves the function.

    */
    bool shouldContinue = true;
    const int messageSize = 40;
    string desiredOperator = currentOperator, utilityString, errorMessage, stringMessage;
    int currentAnswer = 0;
    int *desiredPipe = internalVariableArray.returnPipe(secondOperand);
    static char desiredMessage[messageSize];
    ssize_t bytesRead;
    timespec myReqs;

    myReqs.tv_nsec = 1000000;

    //Note: secondOperand will always initially be px
    //It must be cleared
    firstOperand = to_string(inputVariableArray.returnValueByIdentifier(firstOperand));
    secondOperand = "";

    //The first time a process is initialized a child must wait until child is
    //added into the parents process list. This wait must happen for all children
    //for the first time.
    while(!parentReady)
            nanosleep(NULL, &myReqs);

    while(shouldContinue)
    {
        //It's likely that at this point the appropiate pipe is unclaimed because this child
        //hasn't gotten in touch with the parent.

        if(testMode && verbose) cout << "Paused at line 642" << endl;
        customPause();

        bytesRead = read(desiredPipe[0],desiredMessage, messageSize);
        if(bytesRead == -1 && testMode)
        {
            errorMessage = strerror(errno);
            cout << errorMessage << endl;
        }
        if(testMode)
        {
            cout << getpid() << " is unpaused." << endl;
            cout << "Message is " << (string) desiredMessage << endl;
        }
        tokenizeMessage(desiredMessage, currentOperator, secondOperand);

        if(testMode)
        {
            cout << "~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~" << endl;
            cout << "From arithmeticSpace:" << endl;
            cout << "Process: " << getpid() << endl;
            cout << "desiredMessage: <" << (string) desiredMessage << ">\n";
            if(testMode && verbose)inputVariableArray.printArrayState();
            if(testMode && verbose)internalVariableArray.printArrayState();
            cout << "firstOperand: " << firstOperand << endl
                 << "currentOperator: " << currentOperator << endl
                 << "secondOperand: " << secondOperand << endl << endl << endl;

        }

        if(firstOperand != "" && currentOperator != "" && secondOperand != "")
        {
            //firstOperand and secondOperand will be an input value at this point
            //Note: that the first and second opernad are strings, hence the stoi
            if(currentOperator == "+")
                currentAnswer = stoi(firstOperand) + stoi(secondOperand);
            if(currentOperator == "-")
                currentAnswer = stoi(firstOperand) - stoi(secondOperand);
            if(currentOperator == "*")
                currentAnswer = stoi(firstOperand) * stoi(secondOperand);
            if(currentOperator == "/")
                currentAnswer = stoi(firstOperand) / stoi(secondOperand);

            firstOperand = to_string(currentAnswer);
            if(testMode)
                cout << "currentAnswer is " << currentAnswer << endl;

            //Tell parent process to continue
            //Update internal_var. Example: "p0=2"
            utilityString = to_string(currentAnswer);
            stringMessage = to_string(getpid())+"="+utilityString+";";
            informParent(stringMessage, desiredPipe);

            clearArray(desiredMessage, messageSize);
        }

        //kill(getppid(), SIGCONT);
        kill(parentPid, SIGUSR2);
    }

    //Note that SIGKILL doesn't have a funciton handler yet.
    kill(getpid(), SIGKILL);
}

void arrowSign(string operandOne, string operatorSign, string operandTwo)
{
    //This funciton will execute what the homework described
    //Simply create another thread, and its respective listeners
    //and store it in appropiate variable
    //Possibilities: a -> px || py -> px. These possibilites are
    //irrelevant, because the rightOperand must be an internal variable
    //We need to check if px is initialized

    int pid = 0, desiredPid, currentPid;
    string desiredMessage = "";
    string utilityString = "";
    const int messageSize = 40;
    static char messageArray[messageSize];
    int *desiredPipe;
    int wstatus, utilityInteger;
    struct sigaction mySigValues, mySigValues1, mySigValues2;
    timespec myReqs;

    myReqs.tv_nsec = 1000000;

    mySigValues.sa_flags = SA_SIGINFO;
    mySigValues.sa_sigaction = SIGUSR1Handler;

    mySigValues1.sa_flags = SA_SIGINFO;
    mySigValues1.sa_sigaction = sigContinue;

    mySigValues2.sa_flags = SA_SIGINFO;
    mySigValues2.sa_sigaction = SIGUSR2Handler;

    sigaction(SIGUSR1, &mySigValues, NULL);
    sigaction(SIGCONT, &mySigValues1, NULL);
    sigaction(SIGUSR2, &mySigValues2, NULL);

    if(!internalVariableArray.isInitialized(operandTwo))
    {

        //Parent is not ready. Wait for signal
        parentReady = false;
        internalVariableArray.initializeNextPipe();
        if(testMode && verbose) internalVariableArray.printArrayState();
        pid = fork();
        if(pid == -1)
        {
            perror("Error");
        }
        if(pid == 0)
        {
            //Child Process. Will always go first

            //Initialize first internal variable
            desiredPipe = internalVariableArray.returnFirstUnclaimedPipe();
            currentPid = getpid();
            internalVariableArray.updateValue(currentPid, internalVariableArray.returnIndex(operandTwo));
            internalVariableArray.updatePaused(operandTwo, false);
            internalVariableArray.updateInUse(operandTwo, true);

            arithmeticSpace(operandOne, operatorSign, operandTwo);
        }
        if(pid > 0)
        {
            //Parent Process

            //Wait for child
            //The message should be about updating the paused value of the process. Edit: this comment is wrongish
            currentPid = getpid();
            internalVariableArray.updateValue(pid, internalVariableArray.returnIndex(operandTwo));
            internalVariableArray.updatePaused(operandTwo, false);
            internalVariableArray.updateInUse(operandTwo, true);

            //Parent should be ready
            kill(pid,SIGUSR1);
            parentReady = true;


        }
    }
    else
    {
        //It seems that pipe erases messages if they're not read. MUST READ
        desiredPipe = internalVariableArray.returnElement(operandTwo).personalPipe;
        if(testMode && utilityInteger == -1)
        {
            utilityString = strerror(errno);
            cout << "Error Message: " << utilityString << endl;
        }

        //internal variable is stored and it needs to be accessed and alerted
        //Write message
        if(testMode) inputVariableArray.printArrayState();
        if(isInternalVariable(operandOne)) operandOne = to_string(internalVariableArray.returnElement(operandOne).innerValue);
        else operandOne =  to_string(inputVariableArray.returnValueByIdentifier(operandOne));
        if(operatorSign != "") desiredMessage = operatorSign+";"+operandOne+";";
        else
        {
            desiredMessage = operandTwo+"="+operandOne+";";
            handleMessageForParent(desiredMessage);
        }
        utilityInteger = write(desiredPipe[1], desiredMessage.c_str(), desiredMessage.length());
        desiredPid = internalVariableArray.returnValueByIdentifier(operandTwo);
        //if(internalVariableArray.returnElementByValue(desiredPid).paused) kill(desiredPid,SIGCONT);
        if(testMode) cout << "Message <" << desiredMessage << "> sent to " << desiredPid << " through pipe " << desiredPipe << endl;


        //Child might not have even worked enought to enter the loop and pause. Parent will wait
        while(!internalVariableArray.isPaused(operandTwo)) nanosleep(NULL,&myReqs);

        //Child must be customPaused() at this point!!!!
        if(internalVariableArray.isPaused(to_string(desiredPid)))
        {
            //At this point child is paused. Wake it up and
            //give child time to read message

            //~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Child work start~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~//
            kill(desiredPid,SIGCONT);
            internalVariableArray.updatePaused(operandTwo,false);    //We're not waiting for child to tell us it's paused
            currentChildReadMessage = false;

            while(!currentChildReadMessage)
            {
                nanosleep(NULL, &myReqs);
                if(testMode)
                {
                    if(testMode && verbose)cout << "Looping at line 838\n";
                    //internalVariableArray.printArrayState();
                }
            }
        }

        //~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~Child work end~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~//

        //This next message is to update on innnerValue. MUST BE INNERVALUE AT THIS POINT!!!
        read(desiredPipe[0],messageArray,messageSize);
        handleMessageForParent((string) messageArray);
        if(testMode) internalVariableArray.printArrayState();
        utilityString = "";
    }


}
void sigStop(int parameter)
{
    //if(testMode) cout << "Process Stoped" << endl;
}
void sigContinue(int desiredSig, siginfo_t* desiredSigInfo, void* c)
{
    //Precondition: the caller has stopped. The program is run in parallel.
    //The child MUST be in parents process list. MUST, less get a bad_alloc exception
    //The signal must be idemptotent; call it as often as you want

    //One has to be parent the other child. No mystery
    pid_t caller = desiredSigInfo->si_pid;
    pid_t recipient = getpid();
    string parent = "", desiredIdentifier  = "";

    if(caller == parentPid) parent = "caller";
    else parent = "recipient";

    //Parent called
    if(parent == "caller")
    {
        //This is child process then
        if(testMode)cout << "Parent tells " << recipient << " to continue" << endl;
        desiredIdentifier = internalVariableArray.returnIdentifierByValue(recipient);
        internalVariableArray.updatePaused(desiredIdentifier, false);
        internalVariableArray.updateInUse(desiredIdentifier, true);
    }
    //Child called
    if(parent == "recipient")
    {
        //This is the parent process then
        if(testMode) cout << "Child " << caller << " tells parent " << parentPid << " to continue" << endl;
        desiredIdentifier = internalVariableArray.returnIdentifierByValue(caller);
        internalVariableArray.updatePaused(desiredIdentifier,true);
        internalVariableArray.updateInUse(desiredIdentifier, false);
    }

    if(testMode && verbose) internalVariableArray.printArrayState();

}
void SIGUSR1Handler(int a, siginfo_t* b, void* c)
{
    //Likewise, this handler can only be sent by the parent
    parentReady = !parentReady;
    if(testMode)
    {
        if(parentReady)
            cout << "Parent, " << getppid() << ", told child, " << getpid() << ", it's ready" << endl;
        if(!parentReady)
            cout << "Parent, " << getppid() << ", told child, " << getpid() << ", it's not ready" << endl;
    }
}
void SIGUSR2Handler(int desiredSig, siginfo_t* desiredSigInfo, void* z)
{
    //It should always be the opposite.
    //This is the only signal that can change inUse and paused state of children
    //Precondition: this signal can only be used by children to send to parents
    //Therefore, this should be in the parent process

    pid_t desiredPid =  desiredSigInfo->si_pid;
    currentChildReadMessage = true;
    string desiredIdentifier = internalVariableArray.returnIdentifierByValue(desiredPid);
    internalVariableArray.updateWrittenBack(desiredIdentifier);
    if(testMode)cout << "Child " << desiredPid << " has read the message." << endl;
}
bool isReservedWord(string currentToken)
{
    string inputVar = "input_var", internalVar = "internal_var", write = "write";
    string arrow = "->";
    if(currentToken == inputVar || currentToken == internalVar ||
       currentToken == write    || currentToken == arrow)
                return true;

    return false;
}
void analyzeLine(string desiredLine)
{

    //Eventually, this function got to big, so it needed to be its own function
    //This function just analyzes lines in the graph such as a -> p0
    string currentLine = desiredLine, inputVar = "input_var", internalVar = "internal_var", write = "write";
    string arrow = "->";
    string currentToken;
    string reservedWord = "", operandOne = "", desiredOperator = "", operandTwo = "";
	static char lineArray[50];
	static char tokenArray[40];
	static string parameterArray[20];
	static string operandArray[20];
	char currentCharacter;
	int j = 0, k = 0, varIndex = 0, pIndex = 0, parameterIndex = 0, operandIndex = 0;
    bool acceptingParameters = false;

    currentLine.copy(lineArray, currentLine.length());
    for(int i = 0; i < currentLine.length() || lineArray[i] == ';'; i++)
    {
        currentCharacter = lineArray[i];
        if(!isDelimiter(currentCharacter))
        {
            tokenArray[j] = currentCharacter;
            j++;
        }
        if(isDelimiter(currentCharacter))
        {
            if(currentCharacter == '(')
            {
                //At this point reservedWord = "write"
                acceptingParameters = true;
            }

            currentToken = (string) tokenArray;
            clearArray(tokenArray, (sizeof(tokenArray)/sizeof(*tokenArray)));


            //Note that inputVariableArray is already somewhat filled.
            //You only have to update the identifier for that array.
            if(currentToken != "")
            {
                if(reservedWord == inputVar)
                {
                    inputVariableArray.updateIdentifier(currentToken, varIndex);
                    varIndex++;
                }
                else if(reservedWord == internalVar)
                {
                    internalVariableArray.updateIdentifier(currentToken, pIndex);
                    pIndex++;
                }
                else if(reservedWord == arrow)
                {
                    //Let's say when two operands are put into one internal variable
                    //Do we update value inside the struct array

                    //At this point, operandOne is stored in an array,
                    //the arrow has been read, currentToken should be
                    //the second operand.
                    operandTwo = currentToken;

                    if(operandIndex == 1)
                    {
                        operandOne = operandArray[0];

                        operandIndex = 0;
                        operandArray[0] = "";
                    }
                    if(operandIndex == 2)
                    {
                        desiredOperator = operandArray[0];
                        operandOne = operandArray[1];

                        operandIndex = 0;
                        operandArray[0] = "";
                        operandArray[0] = "";
                    }

                    arrowSign(operandOne, desiredOperator, operandTwo);
                }
                else if(reservedWord == write)
                {
                    //The write function will print out the state of the variables.
                    //They must be pretty printed.
                    if(acceptingParameters)
                    {
                        parameterArray[parameterIndex] = currentToken;
                        parameterIndex++;
                        if(currentCharacter == ')')
                        {

                            inputVariableArray.printParameters(parameterArray);
                            internalVariableArray.printParameters(parameterArray);
                        }

                    }

                }
                else
                {
                    //Note that if we have a token and it isn't a reserved word
                    //then it must be operandOne
                    if(!isReservedWord(currentToken))
                    {
                        operandArray[operandIndex] = currentToken;
                        operandIndex++;
                    }
                }

                if(isReservedWord(currentToken))
                    reservedWord = currentToken;

                clearArray(tokenArray, (sizeof(tokenArray)/sizeof(*tokenArray)));
                j=0;
            }
        }

    }

    clearArray(lineArray, (sizeof(lineArray)/sizeof(*lineArray)));
}
void parseFiles(commandLineArguments desiredStruct)
{
    string graphName = desiredStruct.fileName;
    string numbersName = desiredStruct.numbersFile;
	ifstream myGraph(graphName);
	ifstream myNumbers(numbersName);
	string currentLine = "";

	if(testMode)
        cout << desiredStruct.fileName << endl << desiredStruct.numbersFile << endl;

	while(myNumbers.good())
	{
        getline(myNumbers, currentLine);
        if(currentLine != "")
            tokenizeNumbers(currentLine);
	}

	while(myGraph.good())
	{
		getline(myGraph, currentLine);
        if(currentLine != "")
            analyzeLine(currentLine);

	}


	if(testMode && verbose)
	{
        inputVariableArray.printArrayState();
        internalVariableArray.printArrayState();
	}
}

int main(int argc, char **argv)
{
	commandLineArguments myStruct;
	parseCommandLineArguments(myStruct, argv);
	parseFiles(myStruct);


	if(testMode)
        internalVariableArray.printArrayState();

	return 0;
}

/*
    Note that in arrowSign one of two things will happen, either parent function pauses first or
    arithmetic pauses first, if parent pauses first then it's business as usual. If the child happens
    to finish first then it must tell parent that it's free to move forward.

    Note that SIGKILL is idempotent. It only affects the process if it is stopped.

    Note that arithmetic only pauses when it is done;

    pause() will only be called in either arrowSign() or infiniteArithmetic(). It's only used conservatively

    SIGUSR1-from parent to child, to tell the child parent isnt ready

    SIGUSR2-from child to parent, to tell the parent child isnt ready


    functions with stoi():

        handleMessageForParent()    //Line 862

        infiniteArithmetic()    //In arrowSign in child process. It's probably fed duds

        StructArray<T>::returnIdentifierByValue()   //Return identifier is mentioned twice sigContinue and SIG2USRHandler

        tokenizeNumbers() //The error is not there lol the error is here. It was. Line 469



        BE ADVISED: stoi() and to_string are c++11 functions





*/
