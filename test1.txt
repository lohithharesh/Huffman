#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include<conio.h>

using namespace std;


class FileReader
{
private:
	
	string fileName;

public:
	FileReader(string fileName);
	~FileReader(){;}
	vector<char> readFileIntoVector();
	vector<char> fileVector;
};

FileReader::FileReader(string fName)
{
	fileName = fName;
}

vector<char> FileReader::readFileIntoVector()
{
	
	ifstream fin;
	fin.open(fileName.c_str(), ios::binary);
	
	if(!fin)
	{
		cout << "unable to open file\n";
		exit(1);
	}

	char ch;
	while(fin.get(ch))
		fileVector.push_back(ch);
	fin.close();
	
	
	return fileVector;
}


//----------------------------------------------------------------------------


class CharFrequencyCalculator
{
public:	
	CharFrequencyCalculator(vector<char>& charVec);
	~CharFrequencyCalculator();
	void clearVectors();
	void calculateFrequency();
	void calcFreqInCompressed();
	void displayFrequency();
	vector<char> getUniqueChar(){return uniqueChar;}
	vector<int> getCharFrequency(){return charFrequency;}
private:
	vector<char> charVector;
	vector<char> uniqueChar;
	vector<int>	 charFrequency;
};



CharFrequencyCalculator::CharFrequencyCalculator(vector<char>& charVec)
{
	charVector = charVec;
    
	
}

void CharFrequencyCalculator::displayFrequency()
{
	for(unsigned int i = 0; i < uniqueChar.size(); i++)
	{
		cout <<"character:" << uniqueChar[i] << " frequency:" << charFrequency[i] << endl;
	}
}


void CharFrequencyCalculator::calcFreqInCompressed()
{
	string intString;
	int j = 0;
	int i = 0;

	while(1)
	{
		uniqueChar.push_back(' ');
		charFrequency.push_back(0);
		uniqueChar[j] = charVector[i];
		i++;
		
		while(charVector[i] != ',')
		{
			intString.push_back(charVector[i]);
			i++;
		}
		i++;
		charFrequency[j] = atoi(intString.c_str());

		intString.clear();

		if(uniqueChar[j] == '0' && charFrequency[j] == 0)
		{
			uniqueChar.pop_back();
			charFrequency.pop_back();
			break;
		}
			
		j++;

	}
}



void CharFrequencyCalculator::calculateFrequency()
{
	bool isAlreadyIn = false;
	unsigned int i = 0;
	unsigned int j = 0;

	for(i = 0; i < charVector.size(); i++)
	{
		for(j = 0; j < uniqueChar.size(); j++)
		{
			if(charVector[i] == uniqueChar[j])
			{
				charFrequency[j]++;
				isAlreadyIn = true;
				break;
			}
		
		}
		if(!isAlreadyIn)
		{
			uniqueChar.push_back(charVector[i]);
			charFrequency.push_back(1);
		}
		else
			isAlreadyIn = false;
	}
}

void CharFrequencyCalculator::clearVectors()
{
	charVector.clear();
	uniqueChar.clear();
	charFrequency.clear();

}

CharFrequencyCalculator::~CharFrequencyCalculator()
{
	clearVectors();
}

//--------------------------------------------------------------------------------------------



class Node
{
public:
	Node(){isNode = false;}
	~Node(){};
	Node(Node* child1, Node* child2);

	int getFrequency(){return frequency;}
	char getMyChildrenElementChar(int index) { return myChildren[index]->getCharacter();}
	unsigned int getMyChildrenSize(){ return myChildren.size();}
	void setBitVector(vector <bool> bitVect){bitVector = bitVect;}
	void clearBitVector(){bitVector.clear();}
	void setNull(bool bNull){null = bNull;}
	bool getNull(){return null;}
	bool getIsNode(){return isNode;}
	virtual void displayNodeData();
	virtual char getCharacter(){return '\0';}
	virtual Node *searchChar(char target);
	virtual void ConstructBitVector(); 
	vector<bool> getBitVector(){return bitVector;}
	Node *getMyChildrenElement(int index) { return &*myChildren[index];}
	

protected:
	int frequency;
	bool bit;
	bool isNode;
	bool null;
	string printedData;
	vector <bool> bitVector;

private:
	vector <Node*> myChildren;
	

};


Node::Node(Node* child1, Node* child2)
{
	frequency = child1->frequency + child2->frequency; //need the parent to have the children's combine frequency
	bit = true;
	null = false;
	isNode = true;
	child1->bit = 0; //set the bit of the children left = child1 which = 0 and right = child2 which = 1
	child2->bit = 1;
	myChildren.push_back(child1);
	myChildren.push_back(child2);

}
void Node::ConstructBitVector()
{
	for(int unsigned i = 0; i < myChildren.size(); i++)
	{
		myChildren[i]->bitVector = bitVector;
		myChildren[i]->bitVector.push_back(bit); //give the current bit vector to the child
		myChildren[i]->ConstructBitVector();
	}

}
void Node::displayNodeData()
{
	cout << "freq: " << frequency << endl;
	for(int unsigned i = 0; i < myChildren.size(); i++)
	{
		cout << "child" << i << ": "; 
		myChildren[i]->displayNodeData();
		
	}

}

Node* Node::searchChar(char target)
{
	Node *aNode = this;
	if(aNode->getCharacter() == target)
		return this;
	
	for(int unsigned i = 0; i < myChildren.size(); i++)
	{
		aNode = myChildren[i]->searchChar(target);
		if(aNode->getCharacter() == target)
			break;
	}

	return aNode;

}


//-----------------------------------------------------------------------------------


class LeafNode : public Node
{
public:
	LeafNode(){;}
	LeafNode(char character, int frequency);
	LeafNode(char character);
	~LeafNode(){isNode = false;}
	virtual char getCharacter(){return character;}
	virtual void displayNodeData();
	virtual void ConstructBitVector();
	
private:
	char character;
	
};



LeafNode::LeafNode(char uchar, int freq)
{
	character = uchar;
    frequency = freq;
	isNode = false;
}

LeafNode::LeafNode(char uchar)
{
	character = uchar;
}

void LeafNode::ConstructBitVector()
{
	bitVector.push_back(bit); //give its own bit to the bit vector
}

void LeafNode::displayNodeData()
{
	
	cout <<"char:" << character << " freq:" << frequency << endl;

	
	cout << "The bits assigned to this leaf node are: ";

	for(int unsigned i = 1; i < bitVector.size(); i++) //display the bits assigned to the leaves and start on 1 to not include the root node's bit
	{
		cout << bitVector[i];
		
	}
	cout << endl;
	

}

//---------------------------------------------------------------------------------------------------




class HuffmanTree
{
public:
	HuffmanTree(vector<char> charByte, vector<int> frequency);
	~HuffmanTree();
	vector<Node*> *getTree(){return &nodeList;}
	void constructTree();
	void deleteMemory();
private:
	vector<Node*> nodeList;
	vector<Node*> lowestTwoFreqs;
	vector<char> lowestTwoChars;
	vector<char> characterVector;

};


HuffmanTree::HuffmanTree(vector<char> charVector, vector<int> freqVector)
{


	for(unsigned int i = 0; i < charVector.size(); i++)
	{
		nodeList.push_back(new LeafNode(charVector[i], freqVector[i]));
	}
	characterVector = charVector;
	
}

void HuffmanTree::constructTree()
{
	
	
	// set least to first element
	Node *temp = 0 ;
	int leastFrequency = nodeList[0]->getFrequency();
	int leastIndex = 0;
	int loopTwice = 0;
	

	while(nodeList.size() > 1 )
	{
		while(loopTwice < 2) //loop twice to get lowest two
		{
			for(unsigned int i = 0; i < nodeList.size(); i++)
			{

				if(nodeList[i]->getFrequency() < leastFrequency)//if we find a frequency in the list lower then our current least then set it to least
				{						
						leastIndex = i;	
						leastFrequency = nodeList[i]->getFrequency();
				
				} 
			}

	////			//cout << "the least frequncy is: " << leastfrequency << endl;

			    if(nodeList[leastIndex]->getIsNode())
				{
					lowestTwoFreqs.push_back(new Node);
					
				}
				else
				{
					lowestTwoFreqs.push_back(new LeafNode(nodeList[leastIndex]->getCharacter()));
					
				}
				
				*lowestTwoFreqs[lowestTwoFreqs.size() - 1] = *nodeList[leastIndex];

				delete nodeList[leastIndex];
				nodeList.erase(nodeList.begin()+ leastIndex);

			if(nodeList.size() > 0)
			{
				leastFrequency = nodeList[0]->getFrequency();//initialize a few of these guys back
			}
	
			loopTwice++;			
			leastIndex = 0;


		}
			
		nodeList.push_back (new Node(lowestTwoFreqs[lowestTwoFreqs.size() - 1], lowestTwoFreqs[lowestTwoFreqs.size() - 2]));	
     		loopTwice = 0;
	}
		
	nodeList[0]->clearBitVector(); //clear the root node's bit vector so it will not contribute to the leaf node bit assignment
		
	
	for(unsigned int i = 0; i < nodeList.size(); i++)
	{	
		nodeList[i]->ConstructBitVector();
	}

	nodeList[0]->displayNodeData();
}

void HuffmanTree::deleteMemory()
{
	for(unsigned int i = 0; i < nodeList.size(); i++)
	{

		delete nodeList[i];
	}

}

HuffmanTree::~HuffmanTree()
{
	deleteMemory();
	for(unsigned int i = 0; i < lowestTwoFreqs.size(); i++)//destroy list of lowestfreqs
	{
				delete lowestTwoFreqs[i];
	}
}



//---------------------------------------------------



class CompressFile
{
public:
	CompressFile(vector<Node*> *aList, vector<char> fVector, string outputFile,vector<char> characters, vector<int> frequencies);
	void writeDataToFile();//writes the true and bits to file
	void writeTreeDataToFile();//writes the tree into a file
	void writeBitDataToFile();//writes the bits to a file
	void writeBits(vector<bool> bitVector,int i);
	
private:
	vector<Node*> *nodeList;
	vector<char> fileVector;
	vector<char> charVector ;
	vector<int> freqVector ;
	string outputFileName;
	ofstream fout;
	char tempByte;
	int bitCounter;
	int bitPosition;
	static const int bitsInAByte = 8;
	
};



CompressFile::CompressFile(vector<Node*> *aList, vector<char> fVector, string outputFile, vector<char> characters, vector<int> frequencies)
{
	fileVector = fVector;
	outputFileName = outputFile;
	nodeList = aList;
	bitCounter = 0;
	bitPosition = 128;
	charVector = characters;
	freqVector = frequencies;
}

void CompressFile::writeDataToFile()
{
	
	fout.open(outputFileName.c_str(), ios::binary);
	
	//diplay tree in file
	for(unsigned int i = 0; i < charVector.size(); i++)
	{
		fout << charVector[i];
		fout << freqVector[i];
		fout << ',';	

	} 
	fout << "00" << ','; //marker to mark the end of the tree in the file


	
	writeBitDataToFile();
	

	fout.close();

}


void CompressFile::writeBits(vector<bool> bitVector,int index)
{
		

		for(unsigned int i = 1; i < bitVector.size(); i++)
		{
		if(bitVector[i])
		{
			for(int j = 0; j < bitCounter; j++)
			{
				bitPosition = bitPosition/2;
			}
			tempByte = tempByte | bitPosition;
			bitPosition = 128;
			bitCounter++;
	
		}
		else
		{
			bitCounter++;
		}
		if(bitCounter == 8 || (index == fileVector.size() - 1) && (i == bitVector.size() - 1))
		{
			
			fout << tempByte;//cout dummy byte to a file
			bitCounter = 0;
			tempByte = tempByte & 0;
		}
	}
	

}



void CompressFile::writeBitDataToFile()
{
	
	bool inChild = false;
	Node *aNode;
	tempByte = tempByte & 0; 

	for(unsigned int i = 0; i < fileVector.size(); i++)
	{
		for(unsigned int j = 0; j < nodeList->size(); j++)
		{
			if(nodeList[0][j]->getMyChildrenSize() > 0)
			{
				aNode = nodeList[0][j]->searchChar(fileVector[i]) ;
				writeBits(aNode->getBitVector(),i);
				
			}
		}
	}
}


//-------------------------------------------------------------------------------

class DecompressFile
{
public:
	DecompressFile(vector<Node*> *aList, vector<char> fVector, string outputFile, unsigned int orgFile);
	void decompress();
	void decompressBits(unsigned int i);

private:
	vector<char> fileVector;
	string outputFileName;
	vector<Node*> *nodeList;
	ofstream fout;
	unsigned int originalFileSize;
};


DecompressFile::DecompressFile(vector<Node*> *aList, vector<char> fVector, string outputFile, unsigned int orgFile)
{
	fileVector = fVector;
	outputFileName = outputFile;
	nodeList = aList;
	originalFileSize = orgFile;
}

void DecompressFile::decompress()
{

	unsigned int i = 0;
	
	while(1)
	{
		i++;
		
		while(fileVector[i] != ',')
			i++;
		i++;
		
		if(fileVector[i] == '0' && fileVector[i+1] == '0')
		{
			i = i + 3;
			decompressBits(i);
			break;
		}

	}

}

void DecompressFile::decompressBits(unsigned int i )
{
	int bitPosition = 128;
	Node *temp = &*nodeList[0][0];
	int bitCounter = 0;
	unsigned int currFileSize = 0;
	vector<bool> bitVector;
	ofstream fout;

	fout.open(outputFileName.c_str(), ios::binary);

	
	for(i; i < fileVector.size();)
	{
		
			for(int j = 0; j < bitCounter; j++)
			{
				bitPosition = bitPosition/2;
			}

			if((fileVector[i] & bitPosition) != bitPosition)
			{
     			if(temp->getMyChildrenSize() > 0)
				{
					temp = temp->getMyChildrenElement(0);

					if(temp->getMyChildrenSize() == 0)
					{
						 if(currFileSize < originalFileSize)
                         {
							fout << temp->getCharacter();
						    temp = &*nodeList[0][0];
						    currFileSize++;
                        }
					}
				}
			}
			else
			{
				if(temp->getMyChildrenSize() > 0)
				{
					temp = temp->getMyChildrenElement(1);

					if(temp->getMyChildrenSize() == 0)
					{
							fout << temp->getCharacter();
                            temp = &*nodeList[0][0];
                            currFileSize++;
					}

				}
			}
			
			bitPosition = 128;
			bitCounter++;

		if(bitCounter == 8)
		{
			bitCounter = 0;
			i++;
		}
	}

	fout.close();

}


//-------------------------------------------------------------------

int main()
{
    vector<char> fileVector;
    vector<int> freqVector;
    vector<char> uniqueCharVector;
    vector<char> cmpCharVector;
    vector<char> cmpFileVector;
    vector<int> cmpFreqVector;
    FileReader inFile("test.docx");
    fileVector = inFile.readFileIntoVector();
    
    CharFrequencyCalculator charFreq(fileVector);
    charFreq.calculateFrequency();
    uniqueCharVector = charFreq.getUniqueChar();
    freqVector = charFreq.getCharFrequency();
    charFreq.displayFrequency();
    
    HuffmanTree hTree(uniqueCharVector,freqVector);
    hTree.constructTree();
    
    CompressFile compressFile(hTree.getTree(), fileVector, "compressed.txt", uniqueCharVector, freqVector);
    compressFile.writeDataToFile();
     
     FileReader cmpFile("compressed.txt");
     cmpFileVector = cmpFile.readFileIntoVector();
     
     CharFrequencyCalculator cmpCharFreq(cmpFileVector);
     cmpCharFreq.calcFreqInCompressed();
     
     cmpCharVector = cmpCharFreq.getUniqueChar();
     cmpFreqVector = cmpCharFreq.getCharFrequency();
    // cout<<"\n decompressed\n\n\n";
    // cmpCharFreq.displayFrequency();
     HuffmanTree cmpHTree(cmpCharVector,cmpFreqVector);
     cmpHTree.constructTree();
     
     DecompressFile decompressor(cmpHTree.getTree(),cmpFileVector,"decompressed.txt", fileVector.size() );
     decompressor.decompress();
      
      
    getch();
    
return 0;    
}
