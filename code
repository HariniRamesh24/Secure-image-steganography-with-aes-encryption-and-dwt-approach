function steganography()
% Create main GUI
main_fig = figure('Position', [100 100 400 200], 'Name', 'Image Steganography');
uicontrol('Style', 'pushbutton', 'Position', [50 50 150 30], 'String', 'Embedding',
'Callback', {@embedding_Callback, main_fig});
uicontrol('Style', 'pushbutton', 'Position', [50 100 150 30], 'String', 'Extraction',
'Callback', {@extraction_Callback, main_fig});
end
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
% ------------------ Executes on selecting button embedding -------------------%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
function embedding_Callback(~, ~, main_fig)
% Close the main GUI window
close(main_fig);
% Create the embedding GUI
embed_fig = figure('Position', [100 100 400 200], 'Name', 'Embedding');
% Define UI controls for the embedding GUI
uicontrol('Style', 'pushbutton', 'Position', [50 100 150 30], 'String', 'Browse Cover
Image', 'Callback', @browse_cover_image_Callback);
uicontrol('Style', 'text', 'Position', [50 150 100 20], 'String', 'Selected Image:');
selected_image_text = uicontrol('Style', 'text', 'Position', [170 150 200 20], 'String', '');
uicontrol('Style', 'text', 'Position', [50 50 100 20], 'String', 'Message:');
message_edit = uicontrol('Style', 'edit', 'Position', [150 50 220 30]);
uicontrol('Style', 'pushbutton', 'Position', [50 10 150 30], 'String', 'Embed', 'Callback',
@embed_Callback);
% Initialize variables
cover_image_path = '';
function browse_cover_image_Callback(~, ~)



[filename, pathname] = uigetfile({'*.jpg;*.png;*.bmp', 'Image Files (*.jpg, *.png,
*.bmp)'}, 'Select Cover Image');
if filename == 0
return;
end
cover_image_path = fullfile(pathname, filename);
set(selected_image_text, 'String', cover_image_path);
end
function embed_Callback(~, ~)
% Check if a cover image has been selected
if isempty(cover_image_path)
disp('Please select a cover image.');
return;
end
% Get the message from the UI control
message = get(message_edit, 'String');
% Check if the message is empty
if isempty(message)
disp('Please enter a message.');
return;
end
% Perform embedding (DWT and AES operations)
perform_embedding(cover_image_path, message);
% Optionally, you can close the embedding GUI window after embedding
close(embed_fig);
end
end
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
% ------------------ Executes on selecting button embed ------------------- %
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

function perform_embedding(cover_image_path, message)



% Generate a random 128-bit key and store it in a struct
S.encryptionKey = randi([0 1], 1, 128);
% Create an instance of the AES class with the encryption key
aesInstance = AES(S.encryptionKey, 'SHA-256');
% Encrypt the secret message using the encrypt method
S.encryptedSecretMessage = aesInstance.encrypt(message);
% Save the key and the encrypted message
save('encrypted_data.mat', '-struct', 'S');
% Display the encrypted message
disp('Encrypted Message:');
disp(S.encryptedSecretMessage);
% Decode the base64-encoded encrypted message using Java's Base64
decodedBytes = java.util.Base64.getDecoder().decode(S.encryptedSecretMessage);
binaryEncryptedMessage = char(decodedBytes');
% Convert the binary data to its numeric representation
encryptedNumericMessage = double(binaryEncryptedMessage);
% Read the cover image
coverImage = imread(cover_image_path);
% Discrete Wavelet Transform (DWT) on cover image
[LL1, LH1, HL1, HH1] = dwt2(coverImage, 'haar');
% Get the number of LH1 coefficients required for embedding the secret message
numCoeffs = numel(LH1);
numBitsNeeded = numel(encryptedNumericMessage);
if numBitsNeeded > numCoeffs
error('Secret message is too large to embed in the cover image.');
end
% Embed each bit of the secret message into the LH1 coefficients
binaryIndex = 1;
for i = 1:size(LH1, 1)
for j = 1:size(LH1, 2)
if binaryIndex > numBitsNeeded



break;
end
% Adjust LH1 coefficient based on the corresponding bit of the secret message
bitToEmbed = encryptedNumericMessage(binaryIndex) - '0'; % Convert from char to
numeric value
LH1(i, j) = LH1(i, j) + bitToEmbed;
binaryIndex = binaryIndex + 1;
end
if binaryIndex > numBitsNeeded
break;
end
end
% Inverse DWT to get the stego image
stegoImage = idwt2(LL1, LH1, HL1, HH1, 'haar');
% Display cover image and stego image
figure;
subplot(1,2,1); imshow(uint8(coverImage)); title('Cover Image');
subplot(1,2,2); imshow(uint8(stegoImage)); title('Stego Image');
% Save stego image
imwrite(uint8(stegoImage), 'stego_image.jpg');
End
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
% ------------------ Executes on selecting button extraction ------------------ %
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

function extraction_Callback(,,main_fig)
% Close main GUI
close(main_fig);
% Create extraction GUI
extract_fig = figure('Position', [100 100 400 200], 'Name', 'Extraction');
uicontrol('Style', 'pushbutton', 'Position', [50 50 150 30], 'String', 'Browse Stego Image',
'Callback', @browse_stego_image_Callback);
uicontrol('Style', 'edit', 'Position', [50 100 250 30], 'Tag', 'stego_image_edit');



uicontrol('Style', 'pushbutton', 'Position', [50 150 150 30], 'String', 'Extract', 'Callback',
@extract_Callback);
function browse_stego_image_Callback(~, ~)
[filename, pathname] = uigetfile({'.jpg;.png;.bmp', 'Image Files (.jpg, *.png, *.bmp)'},
'Select Stego Image');
if filename == 0
return;
end
stego_image_edit = findobj(extract_fig, 'Tag', 'stego_image_edit');
set(stego_image_edit, 'String', fullfile(pathname, filename));
end
function extract_Callback(~, ~)
stego_image_edit = findobj(extract_fig, 'Tag', 'stego_image_edit');
stego_image_path = get(stego_image_edit, 'String');
% Perform extraction (inverse DWT and AES decryption)
perform_extraction(stego_image_path);
end
end
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
% ------------------ Executes on selecting button extract------------------- %
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

function perform_extraction(stego_image_path)
% Read stego image
stego_image = imread(stego_image_path);
% Perform DWT on stego image
[~, LH, ~, ~] = dwt2(double(stego_image), 'haar');
% Round LH coefficients to integers
LH = round(LH);
% Extract embedded bits from LH subband
extracted_bits = mod(LH(:), 2);


% Convert extracted bits to characters
extracted_message = char(extracted_bits' + '0'); % Transpose extracted_bits
% Load the encrypted data
loadedData = load('encrypted_data.mat');
% Retrieve the key and encrypted message from the structure
encryptionKey = loadedData.encryptionKey;
encryptedSecretMessage = loadedData.encryptedSecretMessage;
% Create an instance of the AES class with the encryption key
aesInstance = AES(encryptionKey, 'SHA-256');
% Decrypt the encrypted message
decrypted_message = aesInstance.decrypt(encryptedSecretMessage);
% Create a MATLAB GUI window
fig = uifigure('Name', 'Decrypted Message');
% Create a text area to display the decrypted message
txtArea = uitextarea(fig, 'Value', decrypted_message);
txtArea.Position = [20 20 300 100];
txtArea.FontSize = 12;
% Adjust the size and position of the figure window
fig.Position(3:4) = [350 150];
end
